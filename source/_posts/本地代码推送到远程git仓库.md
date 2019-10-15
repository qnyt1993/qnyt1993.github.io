---
title: 本地代码推送到远程git仓库
categories: [杂记]
tags: [git] 
date: 2019-10-15 16:55:15
---


    # 1. 在远程新建一个代码仓库(如码云,github..)
    # 2. 将本地代码提交
    git init
    git add *
    git commit -am "first init"
    git remote add origin https://gitee.com/an1993/mmall_learning.git(这里替换为需要的仓库地址)
    git branch
    git push -u  origin master
    git pull
    git push -u -f  origin master


执行完以上步骤基本就可以将本地代码推送到远程仓库了


    
