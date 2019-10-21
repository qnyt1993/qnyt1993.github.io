---
title: idea配置less自动编译
tags: [other]
categories: [杂记]
date: 2019-10-21 08:49:05
---

> 参考: [idea配置less自动编译](http://www.bmqy.net/1473.html)

## 1. 电脑安装`node.js`环境；

window下直接上[官网](http://nodejs.cn/download/)下载node.msi文件下载安装即可
安装完成后在命令行执行如下命令表明安装成功

    npm -v
    node -v

![](https://raw.githubusercontent.com/qnyt1993/picture/master/img/2019/10/17/QQ%E6%88%AA%E5%9B%BE20191021085827.png)

##  2. 全局安装`less`

    npm install -g less


## 3. 配置`idea`

#### 3.1 安装`nodejs`插件

打开`idea`→`settings`→`plugins`安装：`nodejs`插件，并按以下步骤进行配置：

![](https://raw.githubusercontent.com/qnyt1993/picture/master/img/2019/10/17/1480587751120337.png)
 安装完毕后重启idea
 
#### 3.2 添加`less`组件 

1. 打开`idea`→`settings`→`Languages & Frameworks`→`Node.js and NPM`；
2. 在打开的面板中点击右侧`+`加号按钮添加需要的`less`组件（如果此处不能添加，请使用npm命令进行全局安装）。

![](https://raw.githubusercontent.com/qnyt1993/picture/master/img/2019/10/17/QQ%E6%88%AA%E5%9B%BE20191021091846.png)


#### 3.3 安装`file watchers`插件

打开`idea`→`settings`→`plugins`安装：`file watchers`插件，并按以下步骤进行设置：

![](https://raw.githubusercontent.com/qnyt1993/picture/master/img/2019/10/17/1480587731788935.png)

安装完后重启idea

#### 3.4 配置`file watchers`

1. 打开`idea`→`settings`→`tools`→`file watchers`；

2. 在打开的面板中点击右侧加号按钮添加less配置，貌似插件自动就配置好了

![](https://raw.githubusercontent.com/qnyt1993/picture/master/img/2019/10/17/QQ%E6%88%AA%E5%9B%BE20191021091922.png)


以上步骤成功后，编辑less文件即可自动编译成css文件。

![](https://raw.githubusercontent.com/qnyt1993/picture/master/img/2019/10/17/QQ%E6%88%AA%E5%9B%BE20191021092045.png)
