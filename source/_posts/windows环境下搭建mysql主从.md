---
title: windows环境下搭建mysql主从
categories: [后端,数据库,mysql]
tags: [mysql]
date: 2019-10-28 09:11:31
---

> 参考 [windows环境下mysql主从配置](https://www.cnblogs.com/naruto123/p/8138708.html)

## 1. 环境

|参数|说明|
|----|----|
|主库所在的操作系统|win7|
|主库的版本|mysql-5.6.46-winx64|
|主库的ip地址|127.0.0.1|
|主库的端口|3306|

|参数|说明|
|----|----|
|从库所在的操作系统|win7|
|从库的版本|mysql-5.6.46-winx64|
|从库的ip地址|127.0.0.1|
|从库的端口|3307|

[mysql下载地址](下载地址：https://www.mysql.com/downloads/)

> 主库和从库版本可以一致也可以不一致，需要说明一点，如果两者版本不一致，一般主库的版本需要比从库的版本低，这样就可以避免由于版本问题，有些sql不能执行的问题。

## 2. 数据库安装

下载的是zip包的mysql 将其解压到本机即可

![](https://raw.githubusercontent.com/qnyt1993/picture/master/img/2019/10/17/QQ%E6%88%AA%E5%9B%BE20191028092215.png)

### 2.1 主库（master）的安装及配置

进入主库mysql-5.6.46-winx64目录中，在此目录中新建`my.ini`文件并添加一下配置。


    
       [mysqld]
       
       # 以下内容手动添加
       [client]
       port=3307
       default-character-set=utf8
       [mysqld]
       #主库配置
       server_id=1
       log_bin=master-bin
       log_bin-index=master-bin.index
       
       # 跳过密码
       skip-grant-tables
       
       #端口
       port=3306
       character_set_server=utf8
       #解压目录
       basedir=D:\program\mysql-5.6.46-winx64
       #解压目录下data目录
       datadir=D:\program\mysql-5.6.46-winx64\data
       
       sql_mode=NO_ENGINE_SUBSTITUTION,STRICT_TRANS_TABLES
       [WinMySQLAdmin]
       D:\program\mysql-5.6.46-winx64\bin\mysqld.exe
   


> 将里面的路径修改成你自己的主库路径

以管理员权限打开`cmd`

    cd /d D:\program\mysql-5.6.46-winx64\bin 
    mysqld --install master --defaults-file="D:\program\mysql-5.6.46-winx64\my.ini"
    
    
出现以下提示，表示服务安装成功。

![](https://raw.githubusercontent.com/qnyt1993/picture/master/img/2019/10/17/QQ%E6%88%AA%E5%9B%BE20191028093401.png)

> 将里面的路径修改成你自己的主库路径<br/>  
> 其中的master为主库mysql的服务名称<br/>  
> 删除master服务用sc delete master
 

启动主库的mysql服务器

    net start master

> net stop master 为停止命令

![](https://raw.githubusercontent.com/qnyt1993/picture/master/img/2019/10/17/QQ%E6%88%AA%E5%9B%BE20191028093656.png)

使用命令` >mysql -uroot -P3306 -p `登录`master`数据库（默认安装好的mysql的root用户是没有密码的）

![](https://raw.githubusercontent.com/qnyt1993/picture/master/img/2019/10/17/QQ%E6%88%AA%E5%9B%BE20191028100818.png)

　登录上之后修改root用户的密码（这里修改成root）

执行命令


    　　use mysql;
    　　update  user set password=password("root") where user="root";
    　　flush privileges;


![](https://raw.githubusercontent.com/qnyt1993/picture/master/img/2019/10/17/QQ%E6%88%AA%E5%9B%BE20191028101003.png)

这样就设置好了root用户的密码了。(记得注销my.ini中的跳过密码配置，并重启master)

![](https://raw.githubusercontent.com/qnyt1993/picture/master/img/2019/10/17/QQ%E6%88%AA%E5%9B%BE20191028101235.png)

### 2.2 从库（slave）的安装与配置　　

进入主库D:\program\mysql-5.6.46-winx64-02目录中，在此目录中新建`my.ini`文件并添加一下配置。

    [mysqld]
    
    # 以下内容手动添加
    [client]
    port=3307
    default-character-set=utf8
    [mysqld]
    #从库配置
    server_id=2
    relay-log-index=slave-relay-bin.index
    relay-log=slave-relay-bin
    
    # 跳过密码
    skip-grant-tables
    
    #端口
    port=3307
    character_set_server=utf8
    #解压目录
    basedir=D:\program\mysql-5.6.46-winx64-02
    #解压目录下data目录
    datadir=D:\program\mysql-5.6.46-winx64-02\data
    
    sql_mode=NO_ENGINE_SUBSTITUTION,STRICT_TRANS_TABLES
    [WinMySQLAdmin]
    D:\program\mysql-5.6.46-winx64-02\bin\mysqld.exe

安装从库服务。


    cd /d D:\program\mysql-5.6.46-winx64-02\bin
    mysqld --install slave --defaults-file="D:\program\mysql-5.6.46-winx64-02\my.ini"
    
 ![](https://raw.githubusercontent.com/qnyt1993/picture/master/img/2019/10/17/QQ%E6%88%AA%E5%9B%BE20191028101840.png)

启动从的mysql服务器

    net start slave
    
![](https://raw.githubusercontent.com/qnyt1993/picture/master/img/2019/10/17/QQ%E6%88%AA%E5%9B%BE20191028101943.png)

同样的登录从库(`mysql -uroot -P3307 -p`)

![](https://raw.githubusercontent.com/qnyt1993/picture/master/img/2019/10/17/QQ%E6%88%AA%E5%9B%BE20191028102101.png)

修改从库root用户的密码为root

    　　use mysql;
    　　update  user set password=password("root") where user="root";
    　　flush privileges;    

![](https://raw.githubusercontent.com/qnyt1993/picture/master/img/2019/10/17/QQ%E6%88%AA%E5%9B%BE20191028102208.png)

注销从库中的my.ini中的跳过密码配置,重启slave服务

![](https://raw.githubusercontent.com/qnyt1993/picture/master/img/2019/10/17/QQ%E6%88%AA%E5%9B%BE20191028102348.png)

　至此，主、从数据库的安装及配置就完成了。

## 3. 关联主库（master）与从库（slave）

上面我们已经把master和slave相关配置文件都已添加，并分别启动了master与slave，现在我们分别登录到master和slave的mysql中，

master的mysql 执行命令 `show master status`查看master的状态

![](https://raw.githubusercontent.com/qnyt1993/picture/master/img/2019/10/17/QQ%E6%88%AA%E5%9B%BE20191028102756.png)

slave的mysql  执行命令 `show slave status`查看slave的状态

![](https://raw.githubusercontent.com/qnyt1993/picture/master/img/2019/10/17/QQ%E6%88%AA%E5%9B%BE20191028102818.png)

我们可以发现 ，master的状态下，生成了一个二进制的日志文件，而slave下是空的，所以我们现在就要把主库与从库关联起来。只需要让从库（slave）知道主库（master）的地址就可以了。

　　首先我们需要在主库（master）中创建一个用户用于与从库同步的用户名和密码（这里我创建一个test用户，密码为mysql），并给test用户授权，以用于主库操作从库。


    create user test;
    grant replication slave on *.* to '从库用户名(test)'@'从库主机地址(127.0.0.1)'identified by '密码(mysql)';
    flush privileges;

![](https://raw.githubusercontent.com/qnyt1993/picture/master/img/2019/10/17/QQ%E6%88%AA%E5%9B%BE20191028103132.png)

现在我们切到从库（slave），把主库与从库联系起来。

    change master to master_host='127.0.0.1',master_port=3306,master_user='test',master_password='mysql',master_log_file='master-bin.000001',master_log_pos=0;
    
![](https://raw.githubusercontent.com/qnyt1993/picture/master/img/2019/10/17/QQ%E6%88%AA%E5%9B%BE20191028103256.png)

然后执行命令 `start slave;` 开启主从同步   

![](https://raw.githubusercontent.com/qnyt1993/picture/master/img/2019/10/17/QQ%E6%88%AA%E5%9B%BE20191028103406.png)

后执行命令查看 slave的状态


    show slave status \G; 

出现如下图，则开启主从跟踪成功

![](https://raw.githubusercontent.com/qnyt1993/picture/master/img/2019/10/17/QQ%E6%88%AA%E5%9B%BE20191028103506.png)

## 4. 验证主从同步

我们进入master和slave并查看他们的数据库，如下图：
然后我们在主库中创建一个数据库user，看一下从库有没有变化。

![](https://raw.githubusercontent.com/qnyt1993/picture/master/img/2019/10/17/QQ%E6%88%AA%E5%9B%BE20191028104211.png)
      

至此，主从同步已配置完毕。

> 注意不要往从库中写数据，如果从库写入数据，master_log_pos是不会变化的，主库的信息没有发生变化，当主库又变化和从库一样的操作时就有可能会产生冲突，因此，只能在主库中写数据，从库只能读数据，当然主库也可以读数据。

