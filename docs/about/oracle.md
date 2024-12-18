### 1. not in  (空值)

```
select count(1) from depot where deptno not in (select deptno from emp);
count(1)
---------
0
```

如果子查询`select deptno from emp`中包含空值，not in (空值)返回为空； in (空值) 正常返回数据。

###  2. translate

`translate(expr, from_string, tp_string)` 字符一对一替换，如果to_string为空，则返回空

> 例：提取字符串中的字母 translate(value, '-0123456789', '-')

###  3. 复制表定义及数据

```sql
create table test2 as select * from test;
```

### 3. 一个查询插入不同表

```sql
insert all
into emp1 () values()
into emp2 () values()
select * from test;
```

### 4. connect by

```sql
select level from dual connect by level <= 3;

--拆分字符串
select v.汉字,level from v connect by level <= length(v.汉字)
```

###  5. replace_replace

```sql
--删除不用的字符
selet regext_replace(name, '[AEIOU]') from dual;
--提取数字 '123'
select regexp_replace('中文123', '[^0-9]', '') from dual;
--提取汉子 '中文'
select regexp_replace('中文123', '[0-9]', '') from dual;
```

### 6. regexp_like

```
--只包含字母或数字
with t_basedata as
 (select '中文abc123' as name
    from dual
  union all
  select 'abc123' as name
    from dual)
select * from t_basedata where regexp_like(name, '^[0-9a-zA-Z]+$')

等同关系
regexp_like(name, 'A') --> like '%A%'
regexp_like(name, '^A') --> like 'A%'
regexp_like(name, 'A$') --> like '%A'
regexp_like(name, '^A$') --> like 'A'
regexp_like(name, '16+') --> like '16%'
regexp_like(name, '16*') --> like '1%'
```

##### 正则知识补充

`^`在方括号内，表示否定

 `^`在方括号外表示字符开始

`$`在方括号外表示字符结束

`+`表一次或多次

`*`表零次次或多次

### 7. regexp_substr

```
--提取逗号分割后的第二个单词  '英文'
select regexp_substr('中文,英文,日语', '[^,]+', 1, 2) from dual; 
```

### 8. regexp_count

```
--将逗号分割的字符串改为in列表
select regexp_substr('中文,英文,日语', '[^,]+', 1, level) as v, level
  from dual
connect by level <= regexp_count('中文,英文,日语', ',') + 1;
```

### 9. row_number，rank，dense_rank

`row_number() over(partition by ** order by *) ` 数据相同时，排序值不同，整体连续
`rank() over(partition by ** order by *) ` 数据相同时，排序值相同，整体不连续

`dense_rank() over(partition by ** order by *) ` 数据相同时，排序值相同，整体连续

### 10. 比例函数ratio_to_report

Ratio_to_report() 括号中就是分子，over() 括号中就是分母，分母缺省就是整个占比。

`ratio_to_report(sal) over(partition by deptno)`

### 11. 日期格式处理

```
select trunc(sysdate, 'day') as 周初,
       trunc(sysdate, 'mm') as 月初,
       trunc(sysdate, 'yy') as 年初,
       last_day(sysdate) as 月末,
       add_months(trunc(sysdate, 'mm'), 1) as 下月初,
       to_char(sysdate, 'day') as 周几,
       to_char(sysdate, 'month') as 月份,
       extract(YEAR from systimestamp) as year,
       extract(MONTH from systimestamp) as MONTH,
       extract(DAY from systimestamp) as DAY,
       extract(HOUR from systimestamp) as HOUR,
       extract(MINUTE from systimestamp) as MINUTE,
       extract(SECOND from systimestamp) as SECOND,
       to_char(sysdate, 'd') as d, -- 1表周日，2表周一...
       to_char(sysdate, 'day') as day,--星期日，星期一
       next_day(sysdate, 1) as 下周日,
       to_char(sysdate, 'ww') as ww,--每年1月1号为第一周开始，date+6为第一周
       to_char(sysdate, 'iw') as iw -- 星期一至周日为第一周，因此跨周的话，上年年末周值也为1
  from dual;
  

    周初	2024/6/16 星期日
    月初	2024/6/1 星期六
    年初	2024/1/1 星期一
    月末	2024/6/30 星期日 18:25:57
    下月初	2024/7/1 星期一
    周几	星期日
    月份	6月 
    YEAR	2024
    MONTH	6
    DAY	16
    HOUR	10
    MINUTE	25
    SECOND	57.306
    D	1
    DAY	星期日
    下周日	2024/6/23 星期日 18:25:57
    WW	24
    IW	24

```

