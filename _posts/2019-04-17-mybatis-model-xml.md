---
layout: post
title: "mybatis的查询映射"
date: 2019-04-17
description: "mybatis,xml映射"
tag: java

---
* 目录
{:toc}
### 背景

<img src="/images/posts/mybatis-model-xml/table.png" height="200" width="600">

用sql查询全部组的信息, 及组内元素信息(查询出来的数据如下)

```json
[{
	"id":1,
	"name":"水果",
	"type":"水果",
	"elementList":[{
		"id":1,
		"groupId":1,
		"name":"苹果",
		"toUrl":"www.baidu.com"
	},{
		"id":2,
		"groupId":1,
		"name":"香蕉",
		"toUrl":"www.baidu.com"
	}]
},{
	"id":2,
	"name":"电器",
	"type":"电器",
	"elementList":[{
		"id":1,
		"groupId":1,
		"name":"洗衣机",
		"toUrl":"www.baidu.com"
	}]
}]
```

### 实现思路

现有group,element的实体类, 但是要按上面那种形式展示, 明显需要额外增加一个类, 将两个表关联在一起. 再查询时写好映射关系, 就能按照需要的返回

```java
/**
* group实体类
*/
public class Group{
    private Long id;
    private String name;
    private String type;
    // 省略getter/setter方法
}

/**
* element实体类
*/
public class Element{
    private Long id;
    private Long groupId;
    private String name;
    private String toUrl;
    // 省略getter/setter方法
}

/**
* 额外增加的实体类(继承了Group类)
*/
public class GroupModel extends Group{
   	private List<Element> elementList;
    // 省略getter/setter方法
}
```

### 代码实现

#### controller层

```java
Map<String, Object> params = new HashMap<>();
params.put("groupType", "水果");
List<GroupModel> groupList = elementService.findPageGroups(params);
```

#### elementService层

```java
public List<GroupModel> findGroups(Map<String, Object> params) {
		return elementMapper.findGroups(params);
	}
```

####  elementMapper

```java
List<GroupModel> findGroups(Map<String, Object> params);
```

#### xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd" >
<mapper namespace="com.cl.mapper.ElementMapper">
    <!-- type:返回的pojo类; extends:Group的xml映射,collection是映射到GroupModel里的List<Element> elementList; ofType:list集合属性中pojo的类型 -->
    <resultMap id="GroupMap" type="com.cl.model.GroupModel"
               extends="com.cl.mapper.GroupMapper.BaseResultMap">
        <collection property="ElementList" javaType="java.util.List" ofType="com.cl.domain.Element">
            <id column="e_id" property="id" jdbcType="BIGINT" />
      		<result column="e_group_id" property="groupId" jdbcType="BIGINT" />
            <result column="e_name" property="name" jdbcType="VARCHAR" />
            <result column="e_to_url" property="toUrl" jdbcType="VARCHAR" />
         </collection>
    </resultMap>

<select id="findGroups" resultMap="GroupMap" parameterType="java.util.Map">
        SELECT t1.*,
            t2.id e_id,
    		t2.group_id  as e_group_id,
            t2.name  as e_name,
            t2.to_url  as e_to_url,
        from cl__group t1 left JOIN cl__element t2
        on t1.id=t2.group_id
        where t1.name=#{groupName,jdbcType=VARCHAR} 
    </select>
</mapper>
```

javaType和ofType都是用来指定对象类型的，但是JavaType是用来指定pojo中属性的类型，而ofType指定的是映射到list集合属性中pojo的类型