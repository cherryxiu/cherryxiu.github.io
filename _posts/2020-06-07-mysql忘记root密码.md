---
layout: post
title: "mysql忘记root密码"
date: 2020-06-07
description: "mysql"
tag: mysql

---

* 目录
{:toc}


### 前言

  mysql的本地密码总是容易忘记，明明安装的时候设置得很简单，到用的时候，却总是报1045异常。这时候就需要重置mysql密码啦~

### 具体步骤

1. 关闭正在运行的mysql服务，确保mysql服务要先**关闭**

2. 打开dos窗口，转到`mysql\bin`目录下

3. 输入`mysqld --skip-grant-tables` 回车。`--skip-grant-tables`的意思是启动MySQL服务的时候跳过权限表认证

4. **再开一个DOS窗口**（因为刚才那个DOS窗口已经不能动了），转到`mysql\bin`目录

5. 输入`mysql`回车，如果成功，将出现MySQL提示符 >

6. 连接权限数据库：` use mysql;`(mysql库就是保存用户名的地方)

7. `show tables;`查看所有表，会发现有个user表，这里存放的就是用户名，密码，权限等等账户信息。

8. 输入`select user,host,password from user;` 来查看账户信息。

9. 进入数据库之后使用update语句修改密码

   ```sql
   update user set authentication_string=password('123456') where user='root' and host='localhost'; -- 需要调用mysql的password函数，进行加密 
   ```

10. 之后刷新权限（必须步骤）：

    ```
    flush privileges;
    ```
11. 退出 quit

12. 注销系统，再进入，使用用户名root和刚才设置的新密码登录

> 如果不行，找到mysql的安装位置，打开my.ini文件，在[mysqld]下输入语句：skip-grant-tables，保存即可。

### 参考文件

[忘记 mysql 数据库连接密码（解决方案）](https://blog.csdn.net/weidong_y/article/details/80493743)  

[mysql忘记root密码后，重新设置、修改root密码](https://www.cnblogs.com/dbave/p/12739687.html)
