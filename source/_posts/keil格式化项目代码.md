---
title: keil格式化项目代码
tags: [other]
categories: [杂记]
date: 2019-09-27 21:00:28
---

> 有时候需要用到一个功能，就先会在网上找到对应的程序，但是百度直接拿来的程序通常不是很规范。想着keil5要是有一个自动格式化代码的功能就好啦，上网一查还真有！需要一些设置如下（keil4与keil5都适用）

 
使用`AStyle`进行代码格式化
#### 1. `Astyle` 下载链接 ：链接：`https://share.weiyun.com/5FsV7Ob` 密码：`aqfkk3` 下载并把软件解压
#### 2. `keil5`单击`Tools`菜单--->`Customize Tools Menu`

![QQ截图20190927211321.png](https://i.loli.net/2019/10/29/ifOeS7pPNHbaMQ6.png)

#### 3. 添加`Astyle All Files` 和`Astyle Current File`自定义菜单(可以使用中文)

##### 添加`格式化当前文件`菜单的方法：
1. 新建命令为`格式化当前文件`
2. 添加`Command`命令：单击`...`按钮，选择`Astyle.exe`。
3. Arguments：
   `Astyle Current File`即`格式化当前文件`菜单填写  `!E`
4. 点击`OK`

![QQ截图20190927211604.png](https://i.loli.net/2019/10/29/RUhA1VSWsMnE9NF.png)

##### 添加`格式化project中的所有文件`菜单的方法：
1. 新建命令为`格式化project中的所有文件`
2. 添加`Command`命令：单击`...`按钮，选择`Astyle.exe`。
3. Arguments：
   `Astyle All Files`即`格式化项目所有文件`菜单填写  `"$E*.c" "$E*.h"`
4. 点击`OK`

![QQ截图20190927211535.png](https://i.loli.net/2019/10/29/UEeMSHmPgBCORzf.png)


>注：!E 表示的是当前获得焦点且正在编辑的文件。
    $E*.c和$E*.h代表当前获得焦点且正在编辑文件所在目录下所有.c和.h文件（参考keil uVision的帮助文档）    
    使用的是Astyle默认格式来格式化文件，另外也可以自定义格式，自定义格式参考Astyle的帮助文档。默认格式化后，会备份原文件为源文件名.orig。如果不想让Astyle备份文件，可以使用-n参数。 如：-n !E （表示格式化当前文件，不备份）

 
#### 在`keil`中的使用效果：生成的菜单出现在`Tools`的下拉菜单中，`Astyle`的运行结构出现在`keil`的`Build Output`窗口中。

格式化前 
   
![QQ截图20190927212350.png](https://i.loli.net/2019/10/29/l1gXxd7fIaqShO2.png)

执行格式化命令

![QQ截图20190927212504.png](https://i.loli.net/2019/10/29/3ASkOvXUrZ8b6ce.png)
              
格式化后  

![QQ截图20190927212409.png](https://i.loli.net/2019/10/29/a1NdvDqp27K5eQl.png)



