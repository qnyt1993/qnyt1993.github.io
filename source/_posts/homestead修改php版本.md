---
title: homestead修改php版本
date: 2019-09-20 08:49:10
categories: [后端,php,开发环境]
tags: [homestead,php,ubuntu] 
---


登录后

如果之前没有设置过`root`密码

    sudo passwd root
    
以`root` 权限执行如下命令，选择对应`php`版本

    # 查看所有 php 版本和当前版本
    update-alternatives --display php 
    # 执行后，会列出当前 php 所有版本和编号，输入编号，切换到执行的版本
    update-alternatives --config php 