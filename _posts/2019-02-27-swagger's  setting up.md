---
layout: post
title: "搭建swagger教程"
date: 2019-02-27
description: "swagger-ui"
tag: other
---

​	    Swagger是一个简单又强大的能为你的Restful风格的Api生成文档工具。随着互联网技术的发展，现在的网站架构基本都是前端渲染、后端分离的形态。而前端和后端的唯一联系，变成了API接口，API文档变成了前后端开发人员联系的纽带。在项目中集成这个工具，根据我们自己的配置信息能够自动为我们生成一个api文档展示页，可以在浏览器中直接访问查看项目中的接口信息，同时也可以测试每个api接口。Swagger生成的api文档是实时更新的，你写的api接口有任何的改动都会在文档中及时得表现出来。详情可以参考[官方网站](https://swagger.io/)。
​   
### 前言

效果演示 

<img src="/images/posts/swagger/swagger_show.gif" height="300" width="800"> 

### 准备阶段

项目中的spring版本如下【其它版本可能不适合，有坑的话需要自己慢慢填 ╮（﹀＿﹀）╭ 我也是很懒的，这个security貌似有点影响】：

```xml
<properties>
        	<org.springframework.version>4.2.5.RELEASE</org.springframework.version>
		<activiti.version>5.20.0</activiti.version>
		<spring.security.version>3.2.3.RELEASE</spring.security.version>
		<shiro.version>1.2.3</shiro.version>
		<spring.mybatis.version>1.3.0</spring.mybatis.version>
		<mybatis.version>3.4.0</mybatis.version>
</properties>
```



### 整合步骤

1. 加入 springfox 的 jar包

   ```xml
       <dependency>
           <groupId>io.springfox</groupId>
           <artifactId>springfox-swagger2</artifactId>
           <version>2.2.2</version>
       </dependency>
   
       <dependency>
           <groupId>io.springfox</groupId>
           <artifactId>springfox-swagger-ui</artifactId>
           <version>2.2.2</version>
       </dependency>
   
       <dependency>
           <groupId>io.swagger</groupId>
           <artifactId>swagger-annotations</artifactId>
           <version>1.5.3</version>
       </dependency>
   
       <dependency>
           <groupId>io.springfox</groupId>
           <artifactId>springfox-staticdocs</artifactId>
           <version>2.2.2</version>
           <scope>test</scope>
       </dependency>
   ```

2. web.xml

   ```xml
   	<servlet-mapping>
   		<servlet-name>springServlet</servlet-name>
   		<url-pattern>/swagger-resources</url-pattern>
   	</servlet-mapping>
   	<servlet-mapping>
   		<servlet-name>springServlet</servlet-name>
   		<url-pattern>/v2/api-docs</url-pattern>
   	</servlet-mapping>
   ```

3. 添加 swaggerConfig 配置文件

   ```xml
   package com.**.***.manage;
   
   import org.springframework.context.annotation.Bean;
   import org.springframework.context.annotation.Configuration;
   import org.springframework.web.servlet.config.annotation.EnableWebMvc;
   import springfox.documentation.builders.ApiInfoBuilder;
   import springfox.documentation.builders.PathSelectors;
   import springfox.documentation.builders.RequestHandlerSelectors;
   import springfox.documentation.service.ApiInfo;
   import springfox.documentation.spi.DocumentationType;
   import springfox.documentation.spring.web.plugins.Docket;
   import springfox.documentation.swagger2.annotations.EnableSwagger2;
   
   @EnableWebMvc
   @EnableSwagger2
   @Configuration
   public class SwaggerConfig {
       @Bean
       public Docket createRestApi() {
           return new Docket(DocumentationType.SWAGGER_2)
                   .apiInfo(apiInfo())
                   .select()
                   .apis(RequestHandlerSelectors.basePackage("com.***.***.manage.controller")) // 注意修改此处的包名
                   .paths(PathSelectors.any())
                   .build();
       }
   
       private ApiInfo apiInfo() {
           return new ApiInfoBuilder()
                   .title("接口列表 v1.1.0") // 任意，请稍微规范点
                   .description("接口测试") // 任意，请稍微规范点
                   .termsOfServiceUrl("http://url/swagger-ui.html") // 将“url”换成自己的ip:port(例如"http://localhost:8080/swagger-ui.html")
                   .contact("laowu") // 无所谓（这里是作者的别称）
                   .version("1.1.0")
                   .build();
       }
   }
   ```

4. 为接口添加注解

   ```java
   @Controller
   @Api(value = "广告controller", description = "广告操作")
   public class AdverController extends BaseController {
   
   	@ApiOperation(value = "导入信息", httpMethod = "POST", notes = "导入信息")
   	@RequestMapping(value = "/modules/manage/importUserBlack.htm", method = RequestMethod.POST)
   	public void importUserInfo(
           	@ApiParam(required=true,value="用户文件",name="userInfoFile")@RequestParam(value = "userInfoFile") MultipartFile userInfoFile,
   		@ApiParam(required=true,value="种类",name="type") @RequestParam(value = "type") String type) throws Exception{
   					}
   }
   ```

   > 常用注解：
   >
   > @Api:注解controller，value为@RequestMapping路径
   >
   > @ApiOperation:注解方法，value为简要描述,notes为全面描述,hidden=true Swagger将不显示该方法，默认为false
   >
   > @ApiParam:注解参数,hidden=true Swagger参数列表将不显示该参数,name对应参数名，value为注释，defaultValue设置默认值,allowableValues设置范围值,required设置参数是否必须，默认为false
   >
   > @ApiModel:注解Model
   >
   > @ApiModelProperty:注解Model下的属性，当前端传过来的是一个对象时swagger中该对象的属性注解就是ApiModelProperty中的value
   >

5. 用浏览器输入`<http://localhost:8081/swagger-ui.html#/>`，即可访问 swagger-ui。

### 可能的坑

1. 可能设置swagger-ui.html默认路径，在web-main.xml中添加

   ```xml
   	<mvc:resources mapping="swagger-ui.html" location="classpath:/META-INF/resources/"/>
   	<mvc:resources mapping="/webjars/**" location="classpath:/META-INF/resources/webjars/"/>
   ```

   

### 感谢

- [SpringMVC项目接入Springfox实战遇到的问题集合](https://www.cnblogs.com/fangyuan303687320/p/5066388.html)

- [SpringMVC注解配置Swagger2步骤及相关知识点](https://blog.csdn.net/z28126308/article/details/71126677)

- [如何编写基于OpenAPI规范的API文档](https://blog.csdn.net/xiang__liu/article/details/80396642)

  