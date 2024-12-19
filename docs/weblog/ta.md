## TA 收益计算

### 背景

客户申购赎回基金份额，现需要基于申赎数据，逐笔计算TA收益

### 基础数据

```

select * from koss_qdii_ta_data where fundcode = 'RQF021' and class = 'CLASS A USD (DIST)' and client = 'N00019' and sellercode = 'D00003';
BUSIDATE, FUNDCODE, CLASS, SELLERCODE, CLIENT, SN, BUSITYPE, SHARESID, DTSHARES, DTDIVIDENDPL, AMOUNT, DELIVERAMOUNT, TRADEAMOUNT, AMOUNT_SUB_CHG, AMOUNT_RED_CHG, SHARES, HDCOST, HDPRICE, DIVIDENDPL, CLOSEPL
310 2964  RQF021    CLASS A USD (DIST)  B002  N00019  20161101  20161101  20161107  D00003  3559.55000000 35560.00000000  0.00000000  0.00000000  0.00000000    Excel导入 104     ifoo  2020/4/9 9:29:57  ifoo  2020/4/9 9:29:57  47512496    ops           0                 
311 2965  RQF021    CLASS A USD (DIST)  B002  N00019  20161104  20161104  20161109  D00003  864.86000000  8640.00000000 0.00000000  0.00000000  0.00000000    Excel导入 104     ifoo  2020/4/9 9:29:57  ifoo  2020/4/9 9:29:57  47512496    ops           0                 
312 2966  RQF021    CLASS A USD (DIST)  B002  N00019  20161107  20161107  20161111  D00003  445630.63000000 4451850.00000000  0.00000000  0.00000000  0.00000000    Excel导入 104     ifoo  2020/4/9 9:29:57  ifoo  2020/4/9 9:29:57  47512496    ops           0                 
313 2967  RQF021    CLASS A USD (DIST)  S001  N00019  20161108  20161108  20161115  D00003  4379.00000000 43790.00000000  0.00000000  0.00000000  0.00000000    Excel导入 104     ifoo  2020/4/9 9:29:57  ifoo  2020/4/9 9:29:57  47512496    ops           0                 
314 2968  RQF021    CLASS A USD (DIST)  B002  N00019  20161110  20161110  20161115  D00003  3646.16000000 36170.00000000  0.00000000  0.00000000  0.00000000    Excel导入 104     ifoo  2020/4/9 9:29:57  ifoo  2020/4/9 9:29:57  47512496    ops           0                 
315 2969  RQF021    CLASS A USD (DIST)  S001  N00019  20161111  20161111  20161118  D00003  532.00000000  5250.84000000 0.00000000  0.00000000  0.00000000    Excel导入 104     ifoo  2020/4/9 9:29:57  ifoo  2020/4/9 9:29:57  47512496    ops           0                 

BUSIDATE, FUNDCODE, CLASS, SELLERCODE, CLIENT, SN, BUSITYPE, SHARESID, DTSHARES, DTDIVIDENDPL, AMOUNT, DELIVERAMOUNT, TRADEAMOUNT, AMOUNT_SUB_CHG, AMOUNT_RED_CHG, SHARES, HDCOST, HDPRICE, DIVIDENDPL, CLOSEPL
1 20161101  RQF021  CLASS A USD (DIST)  D00003  N00019  1 B002  47512496  3559.55 0 35560 0 0 35560 1 3559.55 35560 9.99002682923403  0 0
2 20161104  RQF021  CLASS A USD (DIST)  D00003  N00019  2 B002  47512496  864.86  0 8640  0 0 8640  1 4424.41 44200 9.99003256931433  0 0
3 20161107  RQF021  CLASS A USD (DIST)  D00003  N00019  3 B002  47512496  445630.63 0 4451850 0 0 4451850 1 450055.04 4496050 9.99000033418135  0 0
4 20161108  RQF021  CLASS A USD (DIST)  D00003  N00019  4 S001  47512496  4379  0 43790 0 0 0 0.990270078966342 445676.04 4452303.78853662  9.99000033418135  0 43.7885366198765
5 20161110  RQF021  CLASS A USD (DIST)  D00003  N00019  5 B002  47512496  3646.16 0 36170 0 0 36170 1 449322.2  4488473.78853662  9.9894325019699 0 43.7885366198765
6 20161111  RQF021  CLASS A USD (DIST)  D00003  N00019  6 S001  47512496  532 0 5250.84 0 0 0 0.998815994402235 448790.2  4483159.41044557  9.9894325019699 0 -19.7495544281104
```

### 申购计算方式 20161110

```

select 4452303.78853662 + 36170，
4488473.78853662 / 449322.2
 　from dual;
  --4488473.78853662, 9.9894325019699
  
3	91447	B001	IOP	
4	91447	B002	Subscription	

HDCOST 总成本（相当于amount累加） =  昨日成本 + 申购金额(netamount汇总)
HDPRICE 单位成本(会变化)： 总成本 / 总份额
CLOSEPL 申购收益: 无收益
```

### 赎回计算方式 20161108

```
select * from koss_v_qdii_ta_shares where fundcode = 'RQF021' and class = 'CLASS A USD (DIST)' and client = 'N00019' and sellercode = 'D00003';

7	91447	S001	Redemption	
8	91447	S002	Liquidation	
赎回 20161108
HDCOST 4452303.78853662 总成本（相当于按比例减少总成本） =（1- 当前赎回份额 / 前一日总份额） * 前一日总成本
HDPRICE 单位成本(不变)： 总成本 / 当前总份额
CLOSEPL 赎回收益 = 当前赎回的amount - 单位成本 * 赎回份额

select * from koss_v_qdii_ta_shares where fundcode = 'RQF021' and class = 'CLASS A USD (DIST)' and client = 'N00019' and sellercode = 'D00003';

select 1 - 4379 / (3559.55 + 864.86 + 445630.63) as AMOUNT_RED_CHG,
       4496050 * 0.990270078966342 as HDCOST,
       4452303.78853662 / 445676.04 as HDPRICE,
       43790 - 9.99000033418135 * 4379 as CLOSEPL 　from dual;
--AMOUNT_RED_CHG, HDCOST, HDPRICE, CLOSEPL
--0.990270078966342　， 4452303.78853662, 9.99000033418135，43.7885366198684
```

### 计算视图

```
create or replace view koss.koss_v_qdii_ta_shares as
select c.valuationdate as busidate,
       c.fundcode,
       c.class,
       c.sellercode,
       c.client,
       c.sn,
       c.busitype,
       c.sharesid,
       c.shares as dtshares,
       c.dvd as dtdividendpl,
       c.amount,
       c.deliveramount,
       c.tradeamount,
       amount_sub_chg,
       amount_red_chg,
       c.shares_tot as shares,
       c.amount_tot as hdcost,
       decode(c.shares_tot, 0, 0, c.cost_price) as hdprice, --a.ysxsy_tot as closepl ,a.dvd_tot as dividendpl
       sum(c.dvd) over(partition by c.fundcode, c.class, c.client, c.sellercode order by c.valuationdate, c.busitype) as dividendpl, -- dvd_tot总的分红金额
       sum(case
             when c.busitype in ('S001', 'S002') then
             -- 本次赎回收益 =  当前赎回Net Amount - 前一日成本*赎回份额/持有份额
              c.shares * nvl(f_qdii_tanav(c.valuationdate, c.fundcode, c.class), 0) -
              nvl(cost_price, 0) * c.shares
             when c.busitype = 'AT001' then
              c.amount - c.deliveramount - c.tradeamount
             else
              0
           end) over(partition by c.fundcode, c.class, c.client, c.sellercode order by c.valuationdate, c.busitype) as closepl -- ysxsy_tot累计已实现收益
  from (select *
          from (select b.*,
                       --对于申购来说：今日成本 = 昨日成本 + 申购金额
                       case
                         when busitype in ('B001', 'B002') then
                          (amount - tradeamount - deliveramount)
                         else
                          0
                       end as amount_sub_chg,
                       --对于赎回来说：今日成本 = 昨日成本 * (1 - 赎回份额 / 昨日持仓份额)
                       case
                         when busitype in ('S001', 'S002') and
                              (shares_tot - shares_chg) <> 0 then
                          1 - shares / (shares_tot - shares_chg)
                         else
                          1
                       end as amount_red_chg
                  from (select a.fundcode,
                               a.class,
                               a.client,
                               a.sellercode,
                               a.valuationdate,
                               a.shares, --当日交易份额
                               a.sharesid,
                               a.shares * case
                                 when a.busitype in
                                      ('B001', 'B002', 'AT001', 'AT002', 'D002') then
                                  1
                                 when a.busitype in ('S001', 'S002') then
                                  -1
                                 else
                                  0
                               end as shares_chg, --当日份额变动
                               a.amount, --当日交易金额
                               a.tradeamount,
                               a.deliveramount,
                               a.busitype,
                               sum(a.shares * case
                                     when a.busitype in
                                          ('B001', 'B002', 'AT001', 'AT002', 'D002') then
                                      1
                                     when a.busitype in ('S001', 'S002') then
                                      -1
                                     else
                                      0
                                   end) over(partition by a.fundcode, a.class, a.client order by a.valuationdate, a.busitype, a.id) as shares_tot, --持仓份额
                               case
                                 when a.busitype = 'D001' then
                                  (a.amount - a.tradeamount - a.deliveramount)
                                 else
                                  0
                               end as dvd, --当日分红金额
                               row_number() over(partition by a.fundcode, a.class, a.client order by a.valuationdate, a.busitype, a.id) as sn
                          from koss_qdii_ta_data a
                         where 1 = 1
                           and a.busitype in ('B001',
                                              'B002',
                                              'S001',
                                              'S002',
                                              'D001',
                                              'AT001',
                                              'AT002',
                                              'D002')
                           and a.status = '104') b) c model partition by(fundcode, class, client) dimension by(sn) measures(sellercode, valuationdate, shares，sharesid, amount, deliveramount, tradeamount, busitype, shares_tot, dvd, amount_sub_chg, amount_red_chg, 0 amount_tot, 1 cost_price) rules automatic
         order(amount_tot [ sn ] = nvl(amount_tot [ cv(sn) - 1 ], 0) * amount_red_chg [ cv(sn) ] + amount_sub_chg [ cv(sn) ], cost_price [ sn ] = case when shares_tot [ cv(sn) ] = 0 then cost_price [ cv(sn) - 1 ] else amount_tot [ cv(sn) ] / shares_tot [ cv(sn) ] end)) c with read only
;
comment on table KOSS.KOSS_V_QDII_TA_SHARES is '持仓明细视图';
comment on column KOSS.KOSS_V_QDII_TA_SHARES.BUSIDATE is '业务日期';
comment on column KOSS.KOSS_V_QDII_TA_SHARES.FUNDCODE is '组合代码';
comment on column KOSS.KOSS_V_QDII_TA_SHARES.CLASS is '分级';
comment on column KOSS.KOSS_V_QDII_TA_SHARES.SELLERCODE is '销售商代码';
comment on column KOSS.KOSS_V_QDII_TA_SHARES.CLIENT is '持有人';
comment on column KOSS.KOSS_V_QDII_TA_SHARES.SN is '当日交易序号';
comment on column KOSS.KOSS_V_QDII_TA_SHARES.BUSITYPE is '交易类型';
comment on column KOSS.KOSS_V_QDII_TA_SHARES.SHARESID is '来源：seq_qdii_ta_fundshares';
comment on column KOSS.KOSS_V_QDII_TA_SHARES.DTSHARES is '交易份额明细';
comment on column KOSS.KOSS_V_QDII_TA_SHARES.DTDIVIDENDPL is '交易分红';
comment on column KOSS.KOSS_V_QDII_TA_SHARES.AMOUNT is '持仓交易成本';
comment on column KOSS.KOSS_V_QDII_TA_SHARES.DELIVERAMOUNT is '佣金';
comment on column KOSS.KOSS_V_QDII_TA_SHARES.TRADEAMOUNT is '交易费用';
comment on column KOSS.KOSS_V_QDII_TA_SHARES.SHARES is '份额';
comment on column KOSS.KOSS_V_QDII_TA_SHARES.HDCOST is '持仓成本';
comment on column KOSS.KOSS_V_QDII_TA_SHARES.HDPRICE is '持仓价格';
comment on column KOSS.KOSS_V_QDII_TA_SHARES.DIVIDENDPL is '分红收益';
comment on column KOSS.KOSS_V_QDII_TA_SHARES.CLOSEPL is '赎回收益';
```