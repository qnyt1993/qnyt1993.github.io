---
title: docker常用命令
categories: [运维,容器]
tags: [docker,centos7] 
date: 2019-11-04 09:01:13
---

#### 1.  centos6启动docker


    service docker start

#### 2.  centos6设置docker开机自启

    service docker enable

#### 3.  centos7启动docker

    systemctl start docker

#### 4.  centos7设置docker开机自启

    systemctl enable docker

#### 5.  查看docker信息

    docker info

#### 6. centos7检查docker守护进程状态

    systemctl status docker

#### 7. centos7停止docker守护进程

    systemctl stop docker

#### 8. centos7开启docker守护进程 

    systemctl start docker
    
#### 9.  查看当前系统中所有容器列表

    docker ps -a
    
#### 10.  查看当前系统中运行的容器列表  

    docker ps
    
#### 11. 创建带名字的容器

         //带交互会话
         docker run --name 容器名 -i -t 镜像名 /bin/bash 
         //创建守护式容器
         docker run --name 容器名 -d 镜像名  
                     
#### 12. docker删除容器

    docker rm 容器名
    
#### 13. 重新启动已经停止的容器

    docker start 容器名
    docker restart 容器名
    
#### 14.  为运行的容器创建会话

    docker attach 容器名
    
#### 15.  获取守护式容器日志

    docker logs -f 容器名
    //获取日志最后的10条数据
    docker logs --tail 10 容器名
    //获取最新日志
    docker logs  --tail 0 -f  容器名
    //使用-t为日志加上时间戳
    docker logs -ft 容器名
    
#### 16. 查看守护式容器的进程

    docker top 容器名
    
#### 17. 显示容器的统计信息

    docker stats 容器名1 容器名2
    
#### 18.  在容器中运行后台任务

    docker exec -d 容器名 任务
    //demo
    docker exec -d daemon_dave touch /etc/new_config_file
    
#### 19.   为守护进程容器创建交互会话

    docker exec -i -t 容器名 /bin/bash
    
#### 20. 获取容器更多信息

    docker inspect 容器名
    
#### 21.   列出docker镜像

    docker images
    
#### 22.  本地拉取指定版本的镜像

    docker pull ubuntu:12.04
    
#### 23.  查找镜像

    docker search 镜像名
    
#### 24.   登录到dockerhub

    docker login
                 
                     




                  