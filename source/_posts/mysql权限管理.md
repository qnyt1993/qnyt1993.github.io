---
title: mysql权限管理
categories: [运维,database]
tags: [mysql]
date: 2019-10-31 10:01:01
---

#### 1. 创建用户

    CREATE USER username IDENTIFIED BY 'password' ;
    
#### 2. 用户授权

grant授权格式：`grant 权限列表 on 库.表 to 'user_name'@'host name' identified by "密码";`
 
user_name是用户名，host_name为主机名，即用户连接Mysql时所在主机的名字。

    //授予用户增删改查权限
    grant select,insert,update,delete on *.* to 'user'@'%'; 
    //给予全部权限
    grant all privileges on *.* to 'user'@'%' ;
    //限制john用户只有查询cms数据库的权限,可限制到表
    grant select on cms.* to 'john'@'127.0.0.1';
 
每次操作完要刷新授权
 
    flush privileges;
    
#### 3. 查看用户权限

    show grants for '用户'@'ip';
    # show grants for 'root'@'127.0.0.1';

#### 4. 回收权限

 revoke回收权限格式：`revoke 权限列表 on 库.表 from 用户名@'ip';`
 
     revoke select,insert,update,delete ON *.* from 'user'@'%';
     
#### 5. 删除用户

    drop user 'john'@'127.0.0.1';    
    
 
    
    
    
  


       