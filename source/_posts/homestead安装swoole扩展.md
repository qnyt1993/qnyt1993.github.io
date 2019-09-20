---
title: homestead安装swoole扩展
date: 2019-09-20 09:11:41
categories: [后端,php,开发环境]
tags: [ubuntu,swoole,php] 
---


配置好`ubuntu`的国内镜像源并更新
查看`php`版本，并安装对应`php`版本的`dev`

    sudo apt install php7.2-dev
    
配置`pecl`

    sudo pecl channel-update pecl.php.net
    sudo pear clear-cache
    sudo pear update-channels
    sudo pear upgrade
    
安装`php`的`swoole`扩展

    sudo pecl install swoole
    
在`php.ini`中增加`extension=swoole.so`

    php -i | grep php.ini
    
---
    
    vim /etc/php/7.2/cli/php.ini
    
---

    # 在php.ini的尾部增加如下代码
    extension=swoole.so
    
重启`php`

    sudo service php7.2-fpm restart
    
查看`php`是否成功安装`swoole`模块

    php -m | grep swoole
    
补充

    默认数据库账号密码
    账号： homestead 密码：secret
    默认 ssh 账号密码
    账号：vagrant 密码：vagrant
    创建默认 root 用户
    sudo passwd root