---
title: JDBC插入中文数据出现？号地解决问题
tags: [java,jdbc]
categories: [后端,java]
date: 2019-10-11 10:59:39
---

### 1. 查看jdbc配置是否指定编码

    Connection conn = DriverManager.getConnection("jdbc:mysql://localhost:3306/testjdbc","root","123456");

---

在原先的配置上指定编码即可`?characterEncoding=utf8`

---


    Connection conn = DriverManager.getConnection("jdbc:mysql://localhost:3306/testjdbc?characterEncoding=utf8","root","123456");



-----


### 2. 查看mysql数据库及表编码格式是否正常

![](https://raw.githubusercontent.com/qnyt1993/picture/master/img/2019/09/24/5850452-74e7221937f47251.jpg)

上面的即是正确配置，防止中文字符乱码

### 如果不是的话，需要到`my.ini`文件中添加或修改

    [mysqld]
    default_authentication_plugin=mysql_native_password
    port = 3306
    character_set_server = utf8


重启`mysql`