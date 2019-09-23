---
title: git删除远程.idea目录
date: 2019-09-23 21:32:39
categories: [杂记]
tags: [git] 
---

> 软件:git bash
> 已经配置ssh连接github 


#### 1. 进入项目根目录

    $ cd /c/users/john/blog/

#### 2. 删除缓存区`.idea`（保留工作区`.idea`）

    $ git rm --cached -r .idea
    
#### 3. 提交`.gitiginore`文件，将`.idea`从源代码仓库中删除（-m 表示注解）

    $ git commit -m "commit and remove .idea"

#### 4. 推送到远程端
    
    $ git push 