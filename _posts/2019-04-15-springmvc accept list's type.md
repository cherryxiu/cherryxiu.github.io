---
layout: post
title: "springmvc接收list类型"
date: 2019-04-15
description: "springmvc"
tag: springmvc
---

* 目录
{:toc}
### 背景

后台需要接收前端传来的好几条数据, 分别进行修改或保存操作.于是想到用Object实体类封装, 一般一条数据用`@RequestBody Model model`是没有问题的,想着多条数据用`List<Model> model`接收就行,没到并不可以, 就百度总结了一下

### 错误写法

#### 错误写法1

```java
@PostMapping("saveBaseInfo")
public void saveBaseInfo(List<BaseInfo> baseInfo) {
    service.saveBaseInfo(baseInfo);
}
```

#### 错误写法2

```java
@PostMapping("saveBaseInfo")
public void saveBaseInfo(@RequestBody List<BaseInfo> baseInfo) {
    service.saveBaseInfo(baseInfo);
}
```

### 正确写法

#### 1. 对于只有一个`BaseInfo`实体

```java
/**
* BseInfo实体类
*/
public class BaseInfo{
    private String name;
    private String phone;
    // 省略getter/setter方法
}
```



controller

```
@PostMapping("saveBaseInfo")
public void saveBaseInfo(@RequestBody BaseInfo baseInfo) {
    service.saveBaseInfo(baseInfo);
}
```

postman调用

```
{
   "name":"张三",
   "phone":"123"
}
```



#### 2. 对于只有一个`List<BaseInfo> baseInfo`的参数

```java
/**
* BseInfo实体类
*/
public class BaseInfo{
    private String name;
    private String phone;
    // 省略getter/setter方法
}

/**
*BaseInfoModel
*/
public class BaseInfoModel{
   private List<BaseInfo> baseInfoList;
    // 省略getter/setter方法
}
```

controller

```java
@PostMapping("saveBaseInfo")
public void saveBaseInfo(@RequestBody BaseInfoModel baseInfoModel) {
    service.saveBaseInfo(baseInfoModel.getBaseInfoList);
}
```

postman调用

```
{
   "baseInfoList":[{
      "name":"张三",
   	  "phone":"123"
   },{
      "name":"张三",
   	  "phone":"123"
   }]
}
```

#### 3. 对于有两个参数`List<BaseInfo> baseInfo` 和 `String id`

- 参考办法

参考[@RequestBody 和 @RequestParam可以同时使用](https://blog.csdn.net/qq_22076345/article/details/80905460), 下列这种用法我的项目是不可以的,报`Required String parameter 'id' is not present`异常,可能是版本不同的原因

```java
@PostMapping("saveBaseInfo")
public void saveBaseInfo(@RequestBody BaseInfoModel baseInfoModel,
						@RequestParam(value="id") String id) {
    service.saveBaseInfo(baseInfoModel.getBaseInfoList);
}
```

- 解决办法

```java
/**
* BseInfo实体类
*/
public class BaseInfo{
    private String name;
    private String phone;
    // 省略getter/setter方法
}

/**
*BaseInfoModel
*/
public class BaseInfoModel{
   private List<BaseInfo> baseInfoList;
    
    private String  id;
    // 省略getter/setter方法
}
```

controller

```java
@PostMapping("saveBaseInfo")
public void saveBaseInfo(@RequestBody BaseInfoModel baseInfoModel) {
    service.saveBaseInfo(baseInfoModel.getBaseInfoList);
}
```

postman调用

```
{
	"id":"1",
   "baseInfoList":[{
      "name":"张三",
   	  "phone":"123"
   },{
      "name":"张三",
   	  "phone":"123"
   }]
}
```

#### 4. 对于有两个参数`List<BaseInfo> baseInfo` 和 `String id` ,且多层嵌套

```java
/**
* phone实体类
*/
public class Phone{
    private String telphone;
    private String corporation;
    // 省略getter/setter方法
}

/**
* BseInfo实体类
*/
public class BaseInfo{
    private String name;
    private Phone phone;
    // 省略getter/setter方法
}

/**
*BaseInfoModel
*/
public class BaseInfoModel{
   private List<BaseInfo> baseInfoList;
    
    private String  id;
    // 省略getter/setter方法
}
```

controller

```java
@PostMapping("saveBaseInfo")
public void saveBaseInfo(@RequestBody BaseInfoModel baseInfoModel) {
    service.saveBaseInfo(baseInfoModel.getBaseInfoList);
}
```

postman调用

```
{
	"id":"1",
   "baseInfoList":[{
      "name":"张三",
   	  "phone":{
          "telphone":"123",
          "corporation":"移动"
   	  }
   },{
      "name":"张三",
   	  "phone":{
          "telphone":"123",
          "corporation":"移动"
   	  }
   }]
}
```

### postman的使用

<img src="/images/posts/springmvc_accept_list/20190415.png" height="200" width="600"> 



### 感谢

[postman传递list集合后台springmvc接受](https://blog.csdn.net/m0_37034294/article/details/84327937)

