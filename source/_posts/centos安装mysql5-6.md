---
title: centos安装mysql5.6
categories: [运维,数据库,mysql]
tags: [mysql]
date: 2019-10-30 20:25:32
---

> 环境: centos7
> mysql版本: mysql-5.6.46

## 安装mysql5.6

    # 1. 使用rz上传tar包
    # 2. 校验md5值,检查是否和下载页面的md5值一致
    
    md5sum mysql-5.6.46-linux-glibc2.12-x86_64.tar.gz
    
    # 解压
    tar zxf mysql-5.6.46-linux-glibc2.12-x86_64.tar.gz 
    yum install libaio
    groupadd mysql
    useradd -r -g mysql mysql
    cd /usr/local/
    ln -s /root/mysql-5.6.46-linux-glibc2.12-x86_64 mysql
    cd mysql
    chown -R mysql .
    chgrp -R mysql .
    scripts/mysql_install_db --user=mysql
    chown -R root .
    chown -R mysql data
    
    # 启动
    bin/mysqld_safe --user=mysql &
    # 关闭
    mysqladmin shutdown
        
    # another start
    /etc/init.d/mysql.server start
    #         stop
    /etc/init.d/mysql.server stop
   
   
    cp support-files/mysql.server /etc/init.d/mysql.server
    

## 配置开机自启

    # 查看mysql是否开机自启
    chkconfig --list | grep mysql
    # 设置开机自启
    chkconfig --add mysql.server  
    # 配置 /usr/local/mysql/bin路径到环境变量
    vim /etc/profile
    
    # 在底部追加
    export PATH=/usr/local/mysql/bin:$PATH
    
    source /etc/profile
    
    # 查看mysql 版本
    mysql -V
    
    # 设置密码
    set password = password('新密码');
    
## mysql5.7安装脚本

    
    #!/bin/sh
    groupadd mysql
    useradd -r -g mysql mysql
    cd /usr/local
    if [ -d mysql-5.7.9-linux-glibc2.5-x86_64 ]; then 
    echo "mysql folder is exists"
    else
    tar -xzvf  mysql-5.7.9-linux-glibc2.5-x86_64.tar.gz
    fi
    ln -s  mysql-5.7.9-linux-glibc2.5-x86_64 mysql
    cd mysql
    echo "export PATH=/usr/local/mysql/bin:$PATH">>/etc/profile
    source /etc/profile
    service iptables stop
    chkconfig iptables off
    if [ -d mysql-files ]; then
    echo "mysql-files is exists"
    else
    mkdir mysql-files
    fi
    chmod 770 mysql-files
    chown -R mysql .
    chgrp -R mysql .
    if [ -d data ]; then
    mv data data_$(date+%Y%m%d)
    else 
    echo "data is not exist"
    fi
    ./bin/mysqld --initialize --user=mysql
    
    chown -R root .
    chown -R mysql data mysql-files
    ./bin/mysqld_safe --user=mysql &
    
    cp -rf support-files/mysql.server /etc/init.d/mysql.server
    #./usr/local/mysql/support-files/mysql.server stop
    ps -ef|grep mysql|grep -v grep |awk -F' ' '{print $2}'|xargs kill -s 9
           #serivce mysql stop
    ./bin/mysqld_safe --skip-grant-tables &
          #service mysql start
          #./usr/local/mysql/support-files/mysql.server start
    mysql -uroot -p
    use mysql;
    update mysql.user set authentication_string=password('123456') where user='root';
    flush privileges;
    quit;
    
    mysql -uroot -p123456
    set password for 'root'@'localhost'=password("123456");
    flush privileges;




