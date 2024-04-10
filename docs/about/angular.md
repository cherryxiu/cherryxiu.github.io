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

