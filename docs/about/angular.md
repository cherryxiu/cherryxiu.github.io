### 1. 跳转页赋值时, switch的event事件不会触发, 但select, checkbox会触发

### 2. 事后将表单加入校验器

```
     // angualr版本11的写法, 其他版本有其他写法, 参考官网api
     const control: AbstractControl = this.formGroupSubMonitor.get('dispatchid')!;
      if (!control.validator) {
        control.setValidators([control.validator, Validators.required].filter(f => f) as any);
      }
```

### 3. 校验器

~~select 的status = disable,  此时valid = true~~

input 的status = disable,  此时valid = false

### 4. 快速删除node_modules

[原文链接](https://blog.csdn.net/strggle_bin/article/details/110390973)

```
先全局安装 rimraf
npm install -g rimraf
在删除
rimraf node_modules
再重新下载（或者使用镜像cnpm）
npm install
如果在编辑器中删除node_modules文件夹，就使用以下方法

先清除缓存
npm cache clean --force
删除项目中的node_modules文件夹
安装淘宝镜像cnpm，用cnpm来安装依赖(下载的比npm要快)
npm install -g cnpm --registry=https://registry.npm.taobao.org
最后再执行
cnpm install
```

### 创建component

```
加上--spec false为了防止创建测试文件
# ng g c header --spec false
```

### 5. new Set去重 

```
var participantnoList = [...new Set(this.publicTable?.checkedData.map((item: any) => item.participantno))];
```

### 6. 数组合并 [...series1, ...series2];

```
series = [...series1, ...series2];
```

### 7. 深度复制对象

```
import _ from 'lodash';

handleDataset(calendarObj: any[], fundDataObj: any[], participantDataObj: any[]) {
    // 深拷贝
    const calendar = _.cloneDeep(calendarObj);
    const fundData = _.cloneDeep(fundDataObj);
    const participantData = _.cloneDeep(participantDataObj);

}
```

### 8. 比较两对象是否相等 loadsh

```
import _ from 'lodash';


// 表格勾选时系列变化
  ngDoCheck() {
    if (!_.isEqual(this.previousCheckdata, this.publicTable?.checkedData)) {
      // 不相等时的逻辑处理
      }
    }
  }

```

### 9. filter， map， foreach

```
// filter
var fundData = this.publicTable?.checkedData.filter((y: any) => {
    return y.stklongname == x.stklongname && y.participantno == x.participantno;
})

var fundData = this.originFundData.filter((x: any) => stklongnameList.indexOf(x.type) != -1);

// map
calendar.map((calendar: any) => {
            fundData.forEach((fund: any) => {
              if (calendar.busidate == fund.busidate) {
                calendar[fund.type] = Number(fund.amount);
                calendar[fund.type + '-' + 'style'] = { opacity: 0 };
              }
            });
            participantData.forEach((participant: any) => {
              if (calendar.busidate == participant.busidate) {
                calendar[participant.type] = Number(participant.amount);
                calendar[participant.type + '-' + 'stack'] = participant.type.substring(0, participant.type.length - 7);
                calendar[participant.type + '-' + 'style'] = { opacity: 0.3 };
              }
            });
            return calendar;
          })

```
