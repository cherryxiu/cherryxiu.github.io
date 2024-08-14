## bat-windows批处理脚本

### 1. 复制文件

```
chcp 65001
@echo off
echo 复制开始 /y文件重复直接覆盖
::copy \\fundacctnew\QDIIdata\Data\09-21-23\BatchSecurity-ECP.csv \\ia\temp\huihe48\excel\BatchSecurity-ECP.csv
::copy \\fundacctnew\QDIIdata\Data\10-09-23\MarketValue45.csv \\ia\temp\huihe48\wangx26\excel\MarketValue45_1.csv
::copy \\fundacctnew\交易所清算数据\交易所数据\2023\202310\20231009\MarketValue-港股通-20231009.csv \\ia\temp\huihe48\wangx26\excel\MarketValue-港股通-20231009.csv
copy \\p-ir-hkftp\Data\01 报告核对\01 托管行估值报告\CMBWL\BZHKEP\2024\06\18\VALUATION LIST__BZHK01 18Jun2024.xls \\ia\temp\huihe48\wangx26\excel\BZHKEP\2024\06\18\VALUATION LIST__BZHK01 18Jun2024.xls
copy \\p-ir-hkftp\Data\01 报告核对\01 托管行估值报告\CMBWL\BZHKEP\2024\06\19\VALUATION LIST__BZHK01 19Jun2024.xls


pause
```




