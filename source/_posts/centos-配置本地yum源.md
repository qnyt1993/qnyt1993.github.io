---
title: centos6配置本地yum源
categories: [运维,centos]
tags: [yum,centos] 
date: 2019-10-11 11:22:04
---

>在无法访问外网时,`yum`安装软件会失败，这时候可以配置`yum`源为本地的镜像`iso`来解决这个问题 

### 1. 使用`Xftp`上传`iso`镜像文件到服务器

### 2. 使用如下命令新建挂载点并挂载

    sudo mkdir /media/iso
    sudo mkdir /media/dvd1
    sudo mkdir /media/dvd2
    sudo mv  /home/user/CentOS-6.5-x86_64-bin-DVD1.iso   /media/iso
    sudo mv  /home/user/CentOS-6.5-x86_64-bin-DVD2.iso   /media/iso
    #挂载centos安装盘(两个iso)
    sudo mount -o loop /media/iso/CentOS-6.5-x86_64-bin-DVD1.iso  /media/dvd1/
    sudo mount -o loop /media/iso/CentOS-6.5-x86_64-bin-DVD2.iso  /media/dvd2/

### 3. 修改`yum`源配置，把`CentOS-Base.repo`文件备份

    cd /etc/yum.repos.d/
    cp CentOS-Base.repo CentOS-Base.repo.bak

`vim` `CentOS-Base.repo`

    #修改如下内容
    enabled=1
    #将baseurl中的路径修改为 file://media/dvd1和ile://media/dvd2
    #保存退出
    
![](https://i.loli.net/2019/10/29/NlaWG1zYLxeQ4tU.png)


### 4. 清空yum已存在的所有源信息,重新生成缓存

    yum clean all
    yum makecache
    
### 5. 查看本地源的所有软件
       
       yum list    

> 注意系统重启之后，需要再次手动挂载    

至此`Centos`本地`yum`源配置完成    