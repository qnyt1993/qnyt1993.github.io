---
title: 本地管理多个rsa的key
date: 2019-09-19 15:49:02
categories: [杂记]
tags: [rsa,ssh,github] 
description: 关于电脑的多个ssh key管理的文章
---

# 同一台电脑关于多个`SSH KEY`管理

> 笔者之前为电脑中的homestead虚拟机配置过id_rsa，但现在因为想在github上搭建基于hexo的博客，所以需要配置github的ssh key，因此产生需要同一台机器上使用多个SSH key 切换的需求.

### 使用环境

	window7系统

### 环境

	git软件(携带的bash终端类似linux的终端很好用建议安装)
	有一个可用的github账号

### 开始配置(这里仅配置一个，多个类似)

#### 1. 先生成需要的`PUBLIC KEY`

打开`bash`软件执行如下命令

` ssh-keygen -t rsa` 指定生成key的路径名称,一路回车即可


    $ ssh-keygen -t rsa
    Generating public/private rsa key pair.
    Enter file in which to save the key (/c/Users/lenovo/.ssh/id_rsa): /c/Users/lenovo/.ssh/id_rsa_github_hexo
    Enter passphrase (empty for no passphrase):
    Enter same passphrase again:
    Your identification has been saved in /c/Users/lenovo/.ssh/id_rsa_github_hexo.
    Your public key has been saved in /c/Users/lenovo/.ssh/id_rsa_github_hexo.pub.
    The key fingerprint is:
    SHA256:ULsfGWCKY1aJqFQ24QkUxuPu3TiBqoJHXy6mHpQ/i0k lenovo@lenovo-PC
    The key's randomart image is:
    +---[RSA 2048]----+
    |o=o*....+        |
    |.+= +o.+ o       |
    |o..o= o . .      |
    |.. + . . . o     |
    |. +     S o      |
    | +.o  .  . .     |
    |+.E.*o    .      |
    |+o.B++.          |
    |+o=oo.           |
    +----[SHA256]-----+



 这样我们就在`~/.ssh`路径下生成两个文件`id_rsa_github_hexo`和`id_rsa_github_hexo.pub`


    $ ll ~/.ssh
    total 15
    -rw-r--r-- 1 lenovo 197121  114 九月   19 14:32 config
    -rw-r--r-- 1 lenovo 197121 1679 九月   17 14:58 id_rsa
    -rw-r--r-- 1 lenovo 197121  398 九月   17 14:58 id_rsa.pub
    -rw-r--r-- 1 lenovo 197121 1679 九月   19 14:25 id_rsa_github_hexo
    -rw-r--r-- 1 lenovo 197121  398 九月   19 14:25 id_rsa_github_hexo.pub


##### 注意这里


    #设置路径,如果不设置默认生成 id_rsa  和  id_rsa.pub
    Enter file in which to save the key (/root/.ssh/id_rsa):/root/.ssh/id_rsa_aaa  


#### 2. 查看系统ssh-key代理,执行如下命令

执行如下命令查看ssh-key代理

    ssh-add -l

如果如下提示，说明系统代理里没有任何key

    Could not open a connection to your authentication agent.


如果发现上面的提示,,请执行如下操作

    exec ssh-agent bash

如果系统已经有`ssh-key `代理 ,执行下面的命令可以删除


    ssh-add -D

#### 3. 把 .ssh 目录下的新创建的私钥添加的 ssh-agent


    $ ssh-add ~/.ssh/id_rsa_github_hexo
    # 添加成功会有如下提示
    Identity added: /c/Users/lenovo/.ssh/id_rsa_github_hexo (/c/Users/lenovo/.ssh/id_rsa_github_hexo)


#### 4. 打开github的 ssh 管理页面把 对应的公钥提交保存到代码管理服务器 (.pub 结尾)

在终端执行


    cat ~/.ssh/id_rsa_github_hexo.pub

`github`具体位置在 点击账户的`Settings`中的`SSH and FPG keys` 点击`New SSH key`绿色按钮 输入自己本地`id_rsa_github_hexo.pub`中的内容

#### 5. 在 .ssh 目录创建 config 配置文件


    vim ~/.ssh/config


输入如下配置信息(这是配置单个的，多个类似)


    # 配置github 的key
    Host github
        HostName github.com
        User git
        IdentityFile ~/.ssh/id_rsa_github_hexo



#### 6. 测试配置完毕后是否可以ssh访问github


    $ ssh -T git@github.com
    Hi qnyt1993! You've successfully authenticated, but GitHub does not provide shell access.


这里表明已经可以了，配置结束


## 参考

##### [同一台电脑关于多个SSH KEY管理](https://www.cnblogs.com/dfyg-xiaoxiao/p/7281009.html)