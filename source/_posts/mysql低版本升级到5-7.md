---
title: mysql低版本升级到5.7
categories: [运维,database]
tags: [mysql]
date: 2019-10-31 16:08:36
---

## 升级步骤

    #安全的停止数据库的运行
    /etc/init.d/mysql.server stop
    
    # 解压mysql tar包
    tar zxf mysql-5.7.28-linux-glibc2.12-x86_64.tar.gz
    mv mysql-5.7.28-linux-glibc2.12-x86_64 /usr/local/
    cd /usr/local/
    
    # 删除原先软连接,建立新的软连接
    unlink mysql
    ln -s mysql-5.7.28-linux-glibc2.12-x86_64/ mysql
            #此时，MySQL的应用程序版本已经升级完成
            #/etc/init.d/mysqld
            #/etc/profile中PATH增加的/usr/local/mysql/bin
            #都不需要做任何的改变，即可将当前系统的mysql版本升级完成
            #注意：此时只是应用程序升级完成，系统表仍然还是5.6的版本
    cd /usr/local/mysql
    chown root.mysql . -R
    /etc/init.d/mysql.server start 
    
    # 执行更新系统表，输入密码为原先的密码
    mysql_upgrade -p -s
            #参数 -s 一定要加,表示只更新系统表，-s: upgrade-system-tables
            #如果不加-s,则会把所有库的表以5.7.9的方式重建，线上千万别这样操作
            #因为数据库二进制文件是兼容的，无需升级
            #什么时候不需要-s ? 当一些老的版本的存储格式需要新的特性，
            #                 来提升性能时，不加-s
            #即使通过slave进行升级，也推荐使用该方式升级，速度比较快
            
    # 查看升级后的版本
    mysql -V
    # mysql  Ver 14.14 Distrib 5.7.28, for linux-glibc2.12 (x86_64) using  EditLine wrapper

> 5.1.X`、`5.5.X` 、`5.6.X` 是可以直接通过该方式升级到`5.7.X`。`5.0.X`未知，需要测试

## 升级完毕后登录

    [root@localhost mysql]# mysql -uroot -p
    Enter password: 
    Welcome to the MySQL monitor.  Commands end with ; or \g.
    Your MySQL connection id is 3
    Server version: 5.7.28 MySQL Community Server (GPL)
    
    Copyright (c) 2000, 2019, Oracle and/or its affiliates. All rights reserved.
    
    Oracle is a registered trademark of Oracle Corporation and/or its
    affiliates. Other names may be trademarks of their respective
    owners.
    
    Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.
    
    mysql> show databases;
    +--------------------+
    | Database           |
    +--------------------+
    | information_schema |
    | cms                |
    | mysql              |
    | performance_schema |
    | sys                |   # 5.7 新的sys库
    | test               |
    +--------------------+
    6 rows in set (0.00 sec)
    
    mysql> 


         