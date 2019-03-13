---
layout: post
title: "用mybatis-generator-core自动生成pojoExample，mapper和xml"
date: 2019-03-12
description: "mybatis"
tag: other
---
* 目录
{:toc}

看到公司项目中的 mapper 与 xml 中的代码写的十分规范与一致，后来才知道这些是可以通过 mybatis 工具自动生成，倒腾半天，终于可以实现自动生成，以后可以方便码代码啦，哈哈哈......果然，懒惰是推进社会进步的第一生产力。本来 mybatis 的MyBatisCodeHelper-Pro-1.6.6插件也可以实现，但貌似需要花费，就暂时搁置，先讲讲免费的吧，以后在补上。


### 准备阶段
idea，mybatis-genetator-core-1.3.2.jar，mysql-connector-java-5.1.35.jar，generatorConfig.xml   
 [jar包下载](https://pan.baidu.com/s/1WOACd7QUUC6-JfGwBpickQ)

### 开始自动生成
####  1.使用 cmd 执行 jar 包
##### 1.1 放置 jar 包与 xml 文件
不管是 maven 工程还是 web 工程都可以，将两个文件的位置可以随意放置，可以不放在一起（但为了方便操作，我将其都放置在 resources 目录下）
<img src="/images/posts/mybatis-genetator-core/1.png" height="300" width="800"> 
##### 1.2 编辑 generatorConfig.xml文件
`target*`这里都写本地的绝对路径，否则会报错`The specified target project directory src/main/java does not exist`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE generatorConfiguration
  PUBLIC "-//mybatis.org//DTD MyBatis Generator Configuration 1.0//EN"
  "http://mybatis.org/dtd/mybatis-generator-config_1_0.dtd">

<generatorConfiguration>

  <!--
    出现错误：Caused by: java.lang.ClassNotFoundException: com.mysql.jdbc.Driver
    解决办法：将本地的MAVEN仓库中的mysql驱动引入进来
   -->
  <classPathEntry location="C:\Users\wwe\.m2\repository\mysql\mysql-connector-java\5.1.32\mysql-connector-java-5.1.32.jar" />

  <context id="mysqlgenerator" targetRuntime="MyBatis3Simple">
    <jdbcConnection driverClass="com.mysql.jdbc.Driver"
                    connectionURL="jdbc:mysql://127.0.0.1:3306/test"
                    userId="root"
                    password="123456">
    </jdbcConnection>

  <javaTypeResolver>
      <property name="forceBigDecimals" value="false" />
    </javaTypeResolver>

    <javaModelGenerator targetPackage="com.cn.artifact.pojo" targetProject="E:/project/ProjectName/src/main/java">
      <!-- 在targetPackage的基础上，根据数据库的schema再生成一层package，最终生成的类放在这个package下，默认为false -->
      <property name="enableSubPackages" value="true" />
      <!-- 设置是否在getter方法中，对String类型字段调用trim()方法 -->
      <property name="trimStrings" value="true" />
    </javaModelGenerator>

    <!--指定sql映射文件生成的位置 -->
    <sqlMapGenerator targetPackage="config.mappers"  targetProject="E:/project/ProjectName/src/main/resources">
      <property name="enableSubPackages" value="true" />
    </sqlMapGenerator>

    <!-- 指定dao接口生成的位置，mapper接口 -->
    <javaClientGenerator type="XMLMAPPER" targetPackage="com.cn.artifact.mapper"  targetProject="E:/project/ProjectName/src/main/java" >
      <property name="enableSubPackages" value="true" />
    </javaClientGenerator>
    
    <!-- table表生成对应的DoaminObject -->
    <table tableName="t_user" domainObjectName="Tuser" enableSelectByPrimaryKey="true"
      enableUpdateByPrimaryKey="true"
      enableDeleteByPrimaryKey="true"/>
  </context>
</generatorConfiguration>
```
##### 1.3 运行 generatorConfig.xml文件
在 generatorConfig.XML 的所在文件位置，`shift + 右键`，选中`在此处打开命令窗口`,打开 dos 窗口，输入命令
```
java -jar mybatis-generator-core-1.3.2.jar -configfile generatorConfig.xml -overwrite
```
<img src="/images/posts/mybatis-genetator-core/2.png" height="300" width="600"> 
 ps : 若两个文件不在同一目录下，就自行添加 jar 包，xml 的相对路径（相对于打开 dos 窗口的路径）
####  2.使用 maven 的 plugin
##### 2.1 在pom.xml中加入jar包与plugin
```xml
 <plugin>
        <groupId>org.mybatis.generator</groupId>
        <artifactId>mybatis-generator-maven-plugin</artifactId>
        <version>1.3.2</version>
        <dependencies>
          <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>5.1.34</version>
          </dependency>
        </dependencies>
        <configuration>
          <overwrite>true</overwrite>
        </configuration>
      </plugin>
```

右边会出现 mybatis-generator
<img src="/images/posts/mybatis-genetator-core/3.png" height="300" width="600"> 
##### 2.2 修改generatorConfig.xml文件

<img src="/images/posts/mybatis-genetator-core/4.png" height="300" width="600"> 

`target*` 使用相对路径
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE generatorConfiguration
  PUBLIC "-//mybatis.org//DTD MyBatis Generator Configuration 1.0//EN"
  "http://mybatis.org/dtd/mybatis-generator-config_1_0.dtd">

<generatorConfiguration>

  <!--
    出现错误：Caused by: java.lang.ClassNotFoundException: com.mysql.jdbc.Driver
    解决办法：将本地的MAVEN仓库中的mysql驱动引入进来
   -->
  <classPathEntry location="C:\Users\wwe\.m2\repository\mysql\mysql-connector-java\5.1.32\mysql-connector-java-5.1.32.jar" />

  <context id="mysqlgenerator" targetRuntime="MyBatis3Simple">
    <jdbcConnection driverClass="com.mysql.jdbc.Driver"
                    connectionURL="jdbc:mysql://127.0.0.1:3306/test"
                    userId="root"
                    password="123456">
    </jdbcConnection>

  <javaTypeResolver>
      <property name="forceBigDecimals" value="false" />
    </javaTypeResolver>

    <javaModelGenerator targetPackage="com.cn.artifact.pojo" targetProject="src/main/java">
      <!-- 在targetPackage的基础上，根据数据库的schema再生成一层package，最终生成的类放在这个package下，默认为false -->
      <property name="enableSubPackages" value="true" />
      <!-- 设置是否在getter方法中，对String类型字段调用trim()方法 -->
      <property name="trimStrings" value="true" />
    </javaModelGenerator>


    <!--指定sql映射文件生成的位置 -->
    <sqlMapGenerator targetPackage="config.mappers"  targetProject="src/main/resources">
      <property name="enableSubPackages" value="true" />
    </sqlMapGenerator>

    <!-- 指定dao接口生成的位置，mapper接口 -->
    <javaClientGenerator type="XMLMAPPER" targetPackage="com.cn.artifact.mapper"  targetProject="src/main/java" >
      <property name="enableSubPackages" value="true" />
    </javaClientGenerator>

  

    <table tableName="t_user" domainObjectName="Tuser" enableSelectByPrimaryKey="true"
      enableUpdateByPrimaryKey="true"
      enableDeleteByPrimaryKey="true"/>
  </context>
</generatorConfiguration>
```
##### 2.3 双击`mybatis-generator:generate`,运行 plugin

<img src="/images/posts/mybatis-genetator-core/5.png" height="300" width="800"> 

### 最后效果
mapper 文件
```java
package com.cn.artifact.mapper;

import com.cn.artifact.pojo.Tuser;
import java.util.List;

public interface TuserMapper {
    /**
     * This method was generated by MyBatis Generator.
     * This method corresponds to the database table t_user
     *
     * @mbggenerated Sun Jul 29 17:40:22 CST 2018
     */
    int deleteByPrimaryKey(Integer id);

    /**
     * This method was generated by MyBatis Generator.
     * This method corresponds to the database table t_user
     *
     * @mbggenerated Sun Jul 29 17:40:22 CST 2018
     */
    int insert(Tuser record);

    /**
     * This method was generated by MyBatis Generator.
     * This method corresponds to the database table t_user
     *
     * @mbggenerated Sun Jul 29 17:40:22 CST 2018
     */
    Tuser selectByPrimaryKey(Integer id);

    /**
     * This method was generated by MyBatis Generator.
     * This method corresponds to the database table t_user
     *
     * @mbggenerated Sun Jul 29 17:40:22 CST 2018
     */
    List<Tuser> selectAll();

    /**
     * This method was generated by MyBatis Generator.
     * This method corresponds to the database table t_user
     *
     * @mbggenerated Sun Jul 29 17:40:22 CST 2018
     */
    int updateByPrimaryKey(Tuser record);
}
```

xml 文件
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.cn.artifact.mapper.TuserMapper">
  <resultMap id="BaseResultMap" type="com.cn.artifact.pojo.Tuser">
    <!--
      WARNING - @mbggenerated
      This element is automatically generated by MyBatis Generator, do not modify.
      This element was generated on Sun Jul 29 17:40:22 CST 2018.
    -->
    <id column="id" jdbcType="INTEGER" property="id" />
    <result column="username" jdbcType="VARCHAR" property="username" />
    <result column="password" jdbcType="VARCHAR" property="password" />
    <result column="address" jdbcType="VARCHAR" property="address" />
  </resultMap>
  <delete id="deleteByPrimaryKey" parameterType="java.lang.Integer">
    <!--
      WARNING - @mbggenerated
      This element is automatically generated by MyBatis Generator, do not modify.
      This element was generated on Sun Jul 29 17:40:22 CST 2018.
    -->
    delete from t_user
    where id = #{id,jdbcType=INTEGER}
  </delete>
  <insert id="insert" parameterType="com.cn.artifact.pojo.Tuser">
    <!--
      WARNING - @mbggenerated
      This element is automatically generated by MyBatis Generator, do not modify.
      This element was generated on Sun Jul 29 17:40:22 CST 2018.
    -->
    insert into t_user (id, username, password, 
      address)
    values (#{id,jdbcType=INTEGER}, #{username,jdbcType=VARCHAR}, #{password,jdbcType=VARCHAR}, 
      #{address,jdbcType=VARCHAR})
  </insert>
  <update id="updateByPrimaryKey" parameterType="com.cn.artifact.pojo.Tuser">
    <!--
      WARNING - @mbggenerated
      This element is automatically generated by MyBatis Generator, do not modify.
      This element was generated on Sun Jul 29 17:40:22 CST 2018.
    -->
    update t_user
    set username = #{username,jdbcType=VARCHAR},
      password = #{password,jdbcType=VARCHAR},
      address = #{address,jdbcType=VARCHAR}
    where id = #{id,jdbcType=INTEGER}
  </update>
  <select id="selectByPrimaryKey" parameterType="java.lang.Integer" resultMap="BaseResultMap">
    <!--
      WARNING - @mbggenerated
      This element is automatically generated by MyBatis Generator, do not modify.
      This element was generated on Sun Jul 29 17:40:22 CST 2018.
    -->
    select id, username, password, address
    from t_user
    where id = #{id,jdbcType=INTEGER}
  </select>
  <select id="selectAll" resultMap="BaseResultMap">
    <!--
      WARNING - @mbggenerated
      This element is automatically generated by MyBatis Generator, do not modify.
      This element was generated on Sun Jul 29 17:40:22 CST 2018.
    -->
    select id, username, password, address
    from t_user
  </select>
</mapper>
```

### 遇到的问题
1. 只有insert语句，没有selectByPrimarykey语句

[mybatisGenerator生成代码时，只有insert方法](https://blog.csdn.net/gao2419956747/article/details/53197963)
- 1.没有写 `enableUpdateByPrimaryKey="true"`
- 2.数据库的表中没有为id设置主键（我就是这个原因导致的）

2. Caused by: java.lang.ClassNotFoundException: com.mysql.jdbc.Driver

解决办法：将本地的MAVEN仓库中的mysql驱动引入进来

3. 没有Criteria方法生成，没有bySelective方法

将`<context id="mysqlgenerator" targetRuntime="MyBatis3Simple">`改成`<context id="mysqlgenerator" targetRuntime="MyBatis3">`
<img src="/images/posts/mybatis-genetator-core/6.png" height="300" width="600"> 


###  感谢

- [Intellij IDEA中使用MyBatis-generator 自动生成MyBatis代码](https://blog.csdn.net/noaman_wgs/article/details/54409301)

- [通过mybatis工具generatorConfig.xml自动生成实体,DAO,映射文件](https://blog.csdn.net/jinshiyill/article/details/51546676)生成很齐全的文件

- [Mybatis Generator 生成的mapper只有insert方法](https://blog.csdn.net/aitcax/article/details/51078058)只有两个方法的真正原因，数据库的表中没有设置主键