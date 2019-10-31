---
title: mysql5.7单机多实例安装
categories: [运维,database]
tags: [mysql]
date: 2019-10-31 21:41:22
---

> 基于之前的mysql5.7单实例安装

修改`/etc/my.cnf`文件如下(这里配置4个实例，可自行修改数目)

    #
    # 多实例配置文件，可以mysqld_multi --example 查看例子
    #
    [root@MyServer /]> cat /etc/my.cnf 
    #[client]           # 这个标签如果配置了用户和密码，
                        # 并且[mysqld_multi]下没有配置用户名密码，
                        # 则mysqld_multi stop时, 会使用这个密码
                        # 如果没有精确的匹配，则匹配[client]标签
    #user = root        
    #password = 123
    #-------------
    [mysqld_multi]
    mysqld = /usr/local/mysql/bin/mysqld_safe
    mysqladmin = /usr/local/mysql/bin/mysqladmin
    user = multi_admin
    pass = 123  # 官方文档中写的password，但是存在bug，需要改成pass(v5.7.9)
                # 写成password，start时正常，stop时，报如下错误
                # Access denied for user 'multi_admin'@'localhost' (using password: YES)
    log = /var/log/mysqld_multi.log
    
    
    [mysqld1]            
    server-id = 11
    socket = /tmp/mysql.sock1
    port = 3306
    bind_address = 0.0.0.0
    datadir = /data1
    user = mysql
    performance_schema = off
    innodb_buffer_pool_size = 32M
    skip_name_resolve = 1
    log_error = error.log
    pid-file = /data1/mysql.pid1
    
    
    [mysqld2]
    server-id = 12
    socket = /tmp/mysql.sock2
    port = 3307
    bind_address = 0.0.0.0
    datadir = /data2
    user = mysql
    performance_schema = off
    innodb_buffer_pool_size = 32M
    skip_name_resolve = 1
    log_error = error.log
    pid-file = /data2/mysql.pid2
    
    
    [mysqld3]
    server-id = 13
    socket = /tmp/mysql.sock3
    port = 3308
    bind_address = 0.0.0.0
    datadir = /data3
    user = mysql
    performance_schema = off
    innodb_buffer_pool_size = 32M
    skip_name_resolve = 1
    log_error = error.log
    pid-file = /data3/mysql.pid3
    
    
    [mysqld4]
    server-id = 14
    socket = /tmp/mysql.sock4
    port = 3309
    bind_address = 0.0.0.0
    datadir = /data4
    user = mysql
    performance_schema = off
    innodb_buffer_pool_size = 32M
    skip_name_resolve = 1
    log_error = error.log
    pid-file = /data4/mysql.pid4

---

###  准备好数据目录，并初始化安装

    [root@MyServer ~]> mkdir /data1
    [root@MyServer ~]> mkdir /data2
    [root@MyServer ~]> mkdir /data3
    [root@MyServer ~]> mkdir /data4
    [root@MyServer ~]> chown mysql.mysql /data{1..4}
    [root@MyServer ~]> mysqld --initialize --user=mysql --datadir=/data1
    #
    # 一些日志输出，并提示临时密码，下同
    #
    [root@MyServer ~]> mysqld --initialize --user=mysql --datadir=/data2
    [root@MyServer ~]> mysqld --initialize --user=mysql --datadir=/data3
    [root@MyServer ~]> mysqld --initialize --user=mysql --datadir=/data4  
    
    # 安装后，需要检查error.log 确保没有错误出现
    [root@MyServer ~]> cp /usr/local/mysql/support-files/mysqld_multi.server  /etc/init.d/mysqld_multid 
    # 拷贝启动脚本，方便自启
    [root@MyServer ~]> chkconfig mysqld_multid on  
    
    [root@MyServer ~]> mysqld_multi  start
    [root@MyServer ~]> mysqld_multi  report
    Reporting MySQL servers
    MySQL server from group: mysqld1 is running
    MySQL server from group: mysqld2 is running
    MySQL server from group: mysqld3 is running
    MySQL server from group: mysqld4 is running
    [root@MyServer ~]> netstat -tunlp | grep mysql
    [root@MyServer ~]> netstat -tunlp | grep mysql
    tcp        0      0 :::3307                     :::*                        LISTEN      6221/mysqld         
    tcp        0      0 :::3308                     :::*                        LISTEN      6232/mysqld         
    tcp        0      0 :::3309                     :::*                        LISTEN      6238/mysqld         
    tcp        0      0 :::3306                     :::*                        LISTEN      6201/mysqld         
    
    [root@MyServer ~]> mysql -u root -S /tmp/mysql.sock1 -p -P3306
    #
    # 使用-S /tmp/mysql.sock1 进行登录，并输入临时密码后，修改密码，下同
    #
    [root@MyServer ~]> mysql -u root -S /tmp/mysql.sock2 -p -P3307
    [root@MyServer ~]> mysql -u root -S /tmp/mysql.sock3 -p -P3308
    [root@MyServer ~]> mysql -u root -S /tmp/mysql.sock4 -p -P3309
    #输入临时密码,开始修改密码
    
    #MySQL版本5.7.6版本以前用户可以使用如下命令：
    mysql> SET PASSWORD = PASSWORD('123456'); 
    
    #MySQL版本5.7.6版本开始的用户可以使用如下命令：
    mysql> ALTER USER USER() IDENTIFIED BY '123456';  
    
    --
    -- mysql1
    --
    mysql> show variables like "port"; 
    +---------------+-------+
    | Variable_name | Value |
    +---------------+-------+
    | port          | 3306  |
    +---------------+-------+
    1 row in set (0.00 sec)
    
    mysql> show variables like "socket";
    +---------------+------------------+
    | Variable_name | Value            |
    +---------------+------------------+
    | socket        | /tmp/mysql.sock1 |
    +---------------+------------------+
    1 row in set (0.01 sec)
    
    mysql> show variables like "datadir";
    +---------------+---------+
    | Variable_name | Value   |
    +---------------+---------+
    | datadir       | /data1/ |
    +---------------+---------+
    1 row in set (0.00 sec)
       
    --
    -- mysql2, mysql3, mysql4 类似。可以看到与my.cnf中对应的port和socket
    --
    
    ## 关闭数据库(mysqld_multi stop 好像不起作用)
    pkill -9 mysql  
    

> 如果使用    mysqld_multi report 发现有配置的数据库实例没有running ，查看配置datadir中错误日志，如果没有错误日志的话，可以删除datadir中的文件,检查my.cnf中是否有错误，重新执行初始化