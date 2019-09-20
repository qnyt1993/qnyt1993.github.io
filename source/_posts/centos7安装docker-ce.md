---
title: centos7安装docker-ce
date: 2019-09-20 09:19:28
categories: [运维,容器]
tags: [docker,centos7] 
---

给`docker`配置国内的镜像源地址
#### 1. 进入`repos`目录

    cd /etc/yum.repos.d/

#### 2. 获取清华大学的`docker`镜像`repo`

    wget https://mirrors.tuna.tsinghua.edu.cn/docker-ce/linux/centos/docker-ce.repo

#### 3. 修改`docker-ce.repo`改为清华`docker`镜像地址

    vim docker-ce.repo
    
#### 4.下面是使用`vim`执行批量替换(这是清华的`docker` 的`centos7`稳定版的镜像地址`https://mirrors.tuna.tsinghua.edu.cn/docker-ce/linux/centos/7/x86_64/stable/Packages/`)

    :%s@https://download.docker.com/@https://mirrors.tuna.tsinghua.edu.cn/docker-ce/
    
 保存 退出

#### 5. 安装`docker`

    yum install docker-ce

    # 一直选y docker-ce安装结束
    
#### 6. 编辑`docker-ce`的配置文件

    vim /etc/docker/daemon.json
    
#### 7. docker镜像加速

    docker cn
    阿里云加速器
    中国科技大学
    
在`daemon.json`中写入如下`json`,保存退出

    {
        "registry-mirrors" : ["https://registry.docker-cn.com"]
    }
    
启动`docker`并设置开机自启

    systemctl enable docker && systemctl start docker

用`docker info`查看是否配置成功

##### `centos7` `docker`命令

    # 启动docker
    systemctl start docker.service
    # 查看docker 版本
    docker version
    # 查看docker更详细的信息
    docker info
    # 查看当前docker 镜像
    docker image ls
    docker images
    # 搜索镜像
    docker search 镜像名
    # 拉取nginx指定tag的镜像
    docker image pull nginx:1.14-alpine
    # 列出docker所有容器进程
    docker ps
    docker container ls
    # 创建一个有交互且命名的容器
    docker run --name b1 -it busybox:latest
    # 创建一个在后台运行的容器
    docker run --name web1 -d nginx:1.14-alpine
    # 查看容器的信息
    docker inspect 容器名称
    # 启动一个带交互模式的容器
    docker start -it -a 容器名
    # 杀掉一个容器
    docker kill 容器名
    # 查看docker进程(包含杀死的)
    docker ps -a
    # 删除容器
    docker rm 容器名
    # 删除镜像
    docker rmi 镜像名:标签
    # 在指定的容器中执行命令(适合后台运行的容器)
    docker exec 容器名 执行命令
    docker exec -it redis1 /bin/sh
    # 查看容器的日志
    docker contanier logs 容器名
    docker container logs redis1
