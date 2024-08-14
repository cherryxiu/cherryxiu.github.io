## qdii-fe快速格式化代码

### 1. 表格：table.ts DataItem配置

```sql
select '[index: string]: any;'
  from dual
union all
select t.column_name || ': ' || case
         when t.data_type like 'VARCHAR%' then
          'string'
         when t.data_type like 'NUMBER%' then
          'number'
         when t.data_type like 'DATE%' then
          'Date'
       end || '; // ' || t.comments
  from (select lower(a.column_name) as column_name, a.data_type, b.comments
          from user_tab_columns a
          left join user_col_comments b
            on a.table_name = b.table_name
           and a.column_name = b.column_name
         where a.table_name = upper('KOSS_HK_PUBLISH_POSITION')
         order by a.column_id) t;
```

###  2. 表格：table.ts Head配置

> 1.复制到notepad，替换掉："
> 2.若有操作列需单独在第一列写
> 3.特殊列需自己指定width

```sql
select 'new HeadTitle({' || chr(10) || 
       'title: ' || '''' || t.comments || ''',' || chr(10) || 
       'col: ' || '''' || t.column_name || ''',' || chr(10) || 
       'width: ' || '''' || 
       case
         when t.comments like '更新时间' then
          ''
         else
          '100px'
       end || ''',' || chr(10) || 
       case
         when t.comments like '%日期' or t.comments like '组合代码' or
              t.comments like '更新人' or t.comments like '更新时间' then
          'align: ' || '''' || 'center' || ''',' || chr(10)
         else
          ''
       end || 
       case
         when t.comments like '%日期' then
          'isDate: ' || 'true'
         when t.comments like '更新时间' then
          'isSysDate: ' || 'true'
         when t.data_type like 'NUMBER%' then
          'isDecimal: ' || 'true'
       end || '}),'
  from (select lower(a.column_name) as column_name, a.data_type, b.comments
          from user_tab_columns a
          left join user_col_comments b
            on a.table_name = b.table_name
           and a.column_name = b.column_name
         where a.table_name = upper('KOSS_HK_PUBLISH_POSITION')
           and a.column_name not in
               ('ID', 'STKID', 'DELETENO', 'CREATEBY', 'CREATETIME')
         order by a.column_id) t;
```

### 3. 表单：form.ts Form配置

> 1.复制到notepad，替换掉："
> 2.必填的需添加：isRequired: true
> 3.labelSpan，controlSpan默认8

```sql
select case
         when t.comments like '%日期' then
          'new FormDatePicker({'
         when t.comments like '组合代码' then
          'new FormFundPicker({'
         when t.comments like '%币种' or t.comments like '%资产类型%' then
          'new FormSelect({'
         when t.data_type like 'NUMBER%' then
          'new FormInputNumber({'
         else
          'new FormInput({'
       end || chr(10) || 
       'key: ' || '''' || t.column_name || ''',' || chr(10) || 
       case
         when t.comments like '%日期' then
           'value: new Date(),' || chr(10)
         else ''
       end || 
       'label: ' || '''' || t.comments || ''',' || chr(10) ||
       'labelSpan: 8,' || chr(10) || 
       'controlSpan: 8,' || chr(10) || 
       case
         when t.comments like '%日期' then
          'datePicker: {}'
         when t.comments like '组合代码' then
          ''
         when t.comments like '%币种' or t.comments like '%资产类型%' then
          'select: {' || chr(10) || 
          'placeholder: ' || '''请选择'',' || chr(10) || 
          'options: []' || chr(10) || '}'
         when t.data_type like 'NUMBER%' then
          'inputNumber: {' || chr(10) || 
          'placeholder: ' || '''请输入'',' || chr(10) || 
          case when t.comments like '%/%%' escape '/' then 'after: { text: ' || '''%''' || ' },' || chr(10) else '' end  || 
          'min: 0' || chr(10) || '}'
         else
          'input: {' || chr(10) || 
          'placeholder: ' || '''请输入'',' || chr(10) || '}'
       end || '}),'
  from (select lower(a.column_name) as column_name, a.data_type, b.comments
          from user_tab_columns a
          left join user_col_comments b
            on a.table_name = b.table_name
           and a.column_name = b.column_name
         where a.table_name = 'KOSS_HK_PUBLISH_POSITION'
           and a.column_name not in ('ID',
                                     'STKID',
                                     'DELETENO',
                                     'UPDATEBY',
                                     'UPDATETIME',
                                     'CREATEBY',
                                     'CREATETIME')
         order by a.column_id) t;
```

### 4. 表单：ts文件 modal初始化

> 1.多选下拉框的需特殊处理

```sql
select case
         when t.comments like '%日期' then
          t.column_name || ': ' || 'this.formService.toDate(value.' ||
          t.column_name || '),'
         else
          t.column_name || ': ' || 'value.' || t.column_name || ','
       end
  from (select lower(a.column_name) as column_name, a.data_type, b.comments
          from user_tab_columns a
          left join user_col_comments b
            on a.table_name = b.table_name
           and a.column_name = b.column_name
         where a.table_name = 'KOSS_HK_PUBLISH_NAV'
           and a.column_name not in ('ID',
                                     'STKID',
                                     'DELETENO',
                                     'UPDATEBY',
                                     'UPDATETIME',
                                     'CREATEBY',
                                     'CREATETIME')
         order by a.column_id) t;
```

###  5. 表单：ts文件 modal提交

> 1.多选下拉框的需特殊处理

```sql
select case
         when t.column_name = 'id' then
          'id: this.id,'
         else
          t.column_name || ': ' ||
          'this.formService.getValueByKey(this.formExt, this.formGroupExt, ' || '''' ||
          t.column_name || '''),'
       end
  from (select lower(a.column_name) as column_name, a.data_type, b.comments
          from user_tab_columns a
          left join user_col_comments b
            on a.table_name = b.table_name
           and a.column_name = b.column_name
         where a.table_name = 'KOSS_HK_PUBLISH_POSITION'
           and a.column_name not in ('STKID',
                                     'DELETENO',
                                     'UPDATEBY',
                                     'UPDATETIME',
                                     'CREATEBY',
                                     'CREATETIME')
         order by a.column_id) t
union all
select 'userid: this.httpService.getBirUserInfo() ? this.httpService.getBirUserInfo().userID : 0'
  from dual;
```

### 6. 获取表结构

```
select '序号', '名称', '类型', '注释'
  from dual
union all
select to_char(rownum),
       column_name,
       case
         when t.data_type like 'VARCHAR%' then
          data_type || '(' || DATA_LENGTH || ')'
         when t.data_type like 'NUMBER%' then
          data_type || '(' || DATA_PRECISION || ', ' || DATA_SCALE || ')'
         else
          data_type
       
       end,
       t.comments
  from (select lower(a.column_name) as column_name,
               a.data_type,
               b.comments,
               a.DATA_LENGTH,
               a.DATA_PRECISION,
               a.DATA_SCALE
          from user_tab_columns a
          left join user_col_comments b
            on a.table_name = b.table_name
           and a.column_name = b.column_name
         where a.table_name = upper('KOSS_QDII_HKPUBLISH_DATA')
         order by a.column_id) t;
         
```

### 7. 获取查询sql

```
select wm_concat(columnname)
  from (select 't1.' || t.column_name as columnname
          from (select lower(a.column_name) as column_name,
                       a.data_type,
                       b.comments
                  from user_tab_columns a
                  left join user_col_comments b
                    on a.table_name = b.table_name
                   and a.column_name = b.column_name
                 where a.table_name = upper('koss_hk_risk_rule')
                 order by a.column_id) t) a;
```

### 8. 修改

```
select 'update KOSS_QDII_HKSALE_income set '
  from dual
union all
select t.column_name || ' = #{' || t.column_name || ', jdbcType=VARCHAR},' as columnname
  from (select lower(a.column_name) as column_name, a.data_type, b.comments
          from user_tab_columns a
          left join user_col_comments b
            on a.table_name = b.table_name
           and a.column_name = b.column_name
         where a.table_name = upper('KOSS_QDII_HKSALE_income')
         order by a.column_id) t;
```

### 9. 新增

```

select wm_concat(columnname)
  from (select ' ' || t.column_name as columnname
          from (select lower(a.column_name) as column_name,
                       a.data_type,
                       b.comments
                  from user_tab_columns a
                  left join user_col_comments b
                    on a.table_name = b.table_name
                   and a.column_name = b.column_name
                 where a.table_name = upper('koss_qdii_hksale_income')
                 order by a.column_id) t) a;


select '#{' || t.column_name || ', jdbcType=VARCHAR} as '  || t.column_name || ',' as columnname
  from (select lower(a.column_name) as column_name, a.data_type, b.comments
          from user_tab_columns a
          left join user_col_comments b
            on a.table_name = b.table_name
           and a.column_name = b.column_name
         where a.table_name = upper('KOSS_QDII_HKSALE_income')
         order by a.column_id) t;

```