### 利用自定义条件公式进行逻辑判断 sql

#### 背景

​         需要设计一套通用规则，根据页面配置不同的条件，来获取不同的数据。因此将各种不同的判断条件定义为D001, D002, D003等这种，然后前端页面可以调整D001, D002, D003之间的关联关系（使用and ， or这种），来实现不同的逻辑

#### 实现条件 D001 and D002 or D003

`D001 and D002 or D003 `， 每个代表一种条件判断，根据自定义的公式关系，进行条件判断。

```
-- 使用 D001_flag ， D002_flag， D003_flag 为各自标识定义的判断条件结果，符合为1， 不符合为0， 最后根据f_str_calc 结果为1 或 0 判断每条数据是否符合公式要求
select f_str_calc('case when ' || nvl(replace(replace(replace('D001 and D002 or D003',
                                      'D001', c.D001_flag || '=1'),
                                      'D002', c.D002_flag || '=1'),                                          
                                      'D003', c.D003_flag || '=1'),
                                      '1=1') || ' then 1 else 0 end') as flag
  from (select '1' as D001_flag, '0' as D002_flag, '1' as D003_flag
          from dual) c;
```

#### 函数

```
-- 传入函数的执行sql逻辑
select case when 1=1 and 0=1 or 1=1 then 1 else 0 end from dual

create or replace function f_str_calc(exp varchar2) return varchar2 is
  v_exp varchar2(32) := '';
  v_sql varchar2(2000) := '';
  /**************************************************
   计算字符串表达式
  **************************************************/
begin
  v_sql := 'select ' || exp || ' from dual';
  execute immediate v_sql
    into v_exp;
  return v_exp;
end;
```

