## 按FIFO计算数据

### 背景

 有个需求，需要每个季度计算当前持有证券，在本年度中每笔交易的收益。因为同一组合，证券多次买入卖出，而且卖出时按照先买入的顺序依次扣减数量。

1. 如果第一笔交易买入数量 大于 卖出数量，则第一笔交易数量 = 买入数量 - 卖出数量；

2. 如果第一笔交易买入数量 小于 卖出数量，则第一笔买入交易查询不到，并多一条卖出交易，数量= 卖出数量 -第一次买入数量 ；

3. 如果将所有买入完全卖空，则所有买入交易的类型展示为“卖出”。

因为FIFO逻辑在一个sql里实现，较为难理解，特此记录。

### FIFO逻辑

```
with tradeinfo as
 (select t.*,
         sum(t.l_tradeqty) over(partition by t.fundcode, t.stkid, t.bsflag order by t.tradedate, t.tradeid) as l_tradeqty_sum,
         sum(t.tradeqty) over(partition by t.fundcode, t.stkid, t.bsflag order by t.tradedate, t.tradeid) as tradeqty_sum
    from (select a.busiflowid as tradeid,
                 a.settunit as fundcode,
                 a.stkid,
                 to_char(case
                           when a.busitype in ('9922102', '9922103') or
                                a.busitype in ('9932102') and
                                a.orderdate =
                                to_char(to_date(b.DS443, 'yyyy/mm/dd'), 'yyyymmdd') then
                            a.orderdate
                           else
                            a.fundsettdate
                         end) as tradedate,
                 decode(a.busitype,
                        '9932101',
                        '债券买入',
                        '9932102',
                        '债券卖出',
                        '债券兑付') as tradedir,
                 decode(a.busitype, '9932101', 'B', 'S') as bsflag,
                 decode(a.busitype,
                        '9922102',
                        100,
                        '9922103',
                        100,
                        a.matchprice) as tradeprice,
                 a.matchqty as tradeqty,
                 a.matchamt as tradeamt,
                 nvl(lag(a.matchqty, 1)
                     over(partition by a.settunit,
                          a.stkid,
                          decode(a.busitype, '9932101', 'B', 'S') order by
                          a.fundrealsettdate,
                          a.busiflowid),

                        0) as l_tradeqty
                           sum a
                                       left join koss_v_pb_flds_data_bdp b
                                         on a.stkid = b.stkid
                                      where a.busitype in ('9932101', '9932102', '9922102', '9922103')
                                        and a.fundrealsettdate <= '20240731'
                                        and a.status != '2'
                                        and a.stkid = 'FN000010010635') t
                              where t.tradedate <= '20240731'),
                           tradebuy as
                            (select * from tradeinfo a where a.tradedir in ('债券买入')),
                           tradesell as
                            (select * from tradeinfo a where a.tradedir in ('债券卖出', '债券兑付')),
                           tradefifo as
                            (select t.*
                               from (select a.fundcode,
                                            a.stkid,
                                            a.tradedate,
                                            a.tradedir,
                                            a.tradeprice,
                                            case
                                              when b.l_tradeqty_sum >= a.l_tradeqty_sum and
                           b.tradeqty_sum <= a.tradeqty_sum then
                                               b.tradeqty
                                              when b.l_tradeqty_sum < a.l_tradeqty_sum and
                           b.tradeqty_sum <= a.tradeqty_sum then
                                               b.tradeqty_sum - a.l_tradeqty_sum
                                              when b.l_tradeqty_sum >= a.l_tradeqty_sum and
                           b.tradeqty_sum > a.tradeqty_sum then
                                               a.tradeqty_sum - b.l_tradeqty_sum
                                              else
                                               a.tradeqty
                                            end as tradeqty,
                                            b.tradedate as buydate,
                                            b.tradeprice as buyprice
                                       from tradesell a
                                       left join tradebuy b
                                         on a.fundcode = b.fundcode
                                        and a.stkid = b.stkid
                                        and b.tradeqty_sum > a.l_tradeqty_sum
                                        and b.l_tradeqty_sum < a.tradeqty_sum
                                     union all
                                     select a.fundcode,
                                            a.stkid,
                                            a.tradedate,
                                            a.tradedir,
                                            a.tradeprice,
                                            case
                                              when nvl(b.sell_sum, 0) between a.l_tradeqty_sum and
                           a.tradeqty_sum then
                                               a.tradeqty_sum - nvl(b.sell_sum, 0)
                                              else
                                               a.tradeqty
                                            end as tradeqty,
                                            a.tradedate as buydate,
                                            a.tradeprice as buyprice
                                       from tradebuy a
                                       left join (select t0.fundcode,
                                t0.stkid,
                                sum(t0.tradeqty) as sell_sum
                           from tradesell t0
                                                  group by t0.fundcode, t0.stkid) b
                                         on a.fundcode = b.fundcode
                                        and a.stkid = b.stkid
                                      where a.tradeqty_sum > nvl(b.sell_sum, 0)) t
                              order by t.fundcode, t.stkid, t.tradedir, t.tradedate, t.buydate)
                           select * from tradefifo;
```

### 最终全部卖出

卖出交易时按照第一笔买入，依次扣减卖出数量，如果最后一笔交易是全部卖出，则所有买入交易的busitype会变成卖出
原始交易

```
  TRADEID FUNDCODE  STKID TRADEDATE TRADEDIR  BSFLAG  TRADEPRICE  TRADEQTY  TRADEAMT  L_TRADEQTY
1 20220707000026120030  HK0071  FN000010019995  20220711  债券买入  B 96.5  5000.0000 482500.0000 0
2 20230802000026443776  HK0071  FN000010019995  20230804  债券买入  B 97.13 20000.0000  1942600.0000  5000
3 20230802000026443777  HK0071  FN000010019995  20230804  债券买入  B 97.13 10000.0000  971300.0000 20000
4 20240506000058160095  HK0071  FN000010019995  20240506  债券兑付  S 100 35000.0000  3500000.0000  0
```

最终结果

```
  FUNDCODE  STKID TRADEDATE TRADEDIR  TRADEPRICE  TRADEQTY  BUYDATE BUYPRICE
1 HK0071  FN000010019995  20240506  债券兑付  100 5000  20220711  96.5
2 HK0071  FN000010019995  20240506  债券兑付  100 10000 20230804  97.13
3 HK0071  FN000010019995  20240506  债券兑付  100 20000 20230804  97.13
```

### 部分卖出

原始交易

```
  TRADEID FUNDCODE  STKID TRADEDATE TRADEDIR  BSFLAG  TRADEPRICE  TRADEQTY  TRADEAMT L_TRADEQTY
1 20221212000030247222  HK0071  FN000010010635  20221214  债券买入  B 101.6 15000.0000  1524000.0000  0
2 20221212000030247224  HK0071  FN000010010635  20221214  债券买入  B 101.5 10000.0000  1015000.0000  15000
3 20230104000040196036  HK0071  FN000010010635  20230106  债券买入  B 104.89  5000.0000 524450.0000 10000
4 20230104000040196038  HK0071  FN000010010635  20230106  债券买入  B 104.875 10000.0000  1048750.0000  5000
5 20230105000036636002  HK0071  FN000010010635  20230109  债券买入  B 106.125 10000.0000  1061250.0000  10000
6 20230202000025670755  HK0071  FN000010010635  20230206  债券买入  B 108.5 10000.0000  1085000.0000  10000
7 20240522000031184356  HK0071  FN000010010635  20240524  债券买入  B 104.65  10000.0000  1046500.0000  10000
8 20240529000024268689  HK0071  FN000010010635  20240531  债券买入  B 103.98  10000.0000  1039800.0000  10000
9 20230419000033014608  HK0071  FN000010010635  20230421  债券卖出  S 104 10000.0000  1040000.0000  0
10  20230504000056124269  HK0071  FN000010010635  20230508  债券卖出  S 104.9 10000.0000  1049000.0000  10000
```



最终结果

 ```
  FUNDCODE  STKID TRADEDATE TRADEDIR  TRADEPRICE  TRADEQTY  BUYDATE BUYPRICE
 1 HK0071  FN000010010635  20221214  债券买入  101.5 5000  20221214  101.5
 2 HK0071  FN000010010635  20230106  债券买入  104.89  5000  20230106  104.89
 3 HK0071  FN000010010635  20230106  债券买入  104.875 10000 20230106  104.875
 4 HK0071  FN000010010635  20230109  债券买入  106.125 10000 20230109  106.125
 5 HK0071  FN000010010635  20230206  债券买入  108.5 10000 20230206  108.5
 6 HK0071  FN000010010635  20240524  债券买入  104.65  10000 20240524  104.65
 7 HK0071  FN000010010635  20240531  债券买入  103.98  10000 20240531  103.98
 8 HK0071  FN000010010635  20230421  债券卖出  104 10000 20221214  101.6
 9 HK0071  FN000010010635  20230508  债券卖出  104.9 5000  20221214  101.5
 10  HK0071  FN000010010635  20230508  债券卖出  104.9 5000  20221214  101.6
 ```

