---
title: centos7安装最新稳定版nginx
date: 2019-09-20 11:27:56
categories: [运维,nginx]
tags: [nginx,centos7] 
---

开始安装
yum 安装 nginx
#### yum安装nginx文档地址

    # 一切以最新的文档页面为准--搜centos
    http://nginx.org/en/linux_packages.html

#### 安装基础软件
    yum -y install gcc gcc-c++ autoconf pcre pcre-devel make automake
    yum -y install wget httpd-tools vim
    
#### 配置`nginx`的`repo`
    
    cd /etc/yum.repos.d
    vim nginx.repo
    
编辑`nginx.repo`

    [nginx-stable]
    name=nginx stable repo
    baseurl=http://nginx.org/packages/centos/7/$basearch/
    gpgcheck=1
    enabled=1
    gpgkey=https://nginx.org/keys/nginx_signing.key

    [nginx-mainline]
    name=nginx mainline repo
    baseurl=http://nginx.org/packages/mainline/centos/7/$basearch/
    gpgcheck=1
    enabled=0
    gpgkey=https://nginx.org/keys/nginx_signing.key
    
    
重新生成`Yum`缓存

    yum clean all
    yum makecache
    
查看`nginx`是否可以找到

    yum list | grep nginx
    
安装`nginx`

    yum install -y nginx