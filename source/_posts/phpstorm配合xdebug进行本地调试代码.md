---
title: phpstorm配合xdebug进行本地调试代码
date: 2019-09-20 11:15:44
categories: [后端,php,phpstorm]
tags: [php,phpstorm,xdebuge] 
---

>笔者在使用的环境是`wamp3.1.6`和`phpstorm2018` ，`php`选择的环境是`php7.2`

#### 1. 在`php.ini`中添加`xdebug`的配置信息
首先建议是先找对`php.ini`的位置,可以在`phpinfo()`方法中找到`php.ini`文件的位置

![img、1580998-20190312105534466-205518377.png](https://i.loli.net/2019/10/29/LNVHcO8fKnDxu1l.png)


用编辑器打开`php.ini`在末尾追加关于`xdebug`的配置

    [XDebug]
    
    xdebug.profiler_output_dir="D:\Log\xdebug"
    xdebug.trace_output_dir="D:\Log\xdebug"
    xdebug.remote_log="D:/Log/xdebug.log"
    zend_extension="C:/wamp64/bin/php/php7.2.10/zend_ext/php_xdebug-2.6.1-7.2-vc15-x86_64.dll"
    
    ;允许收集传递给函数的参数变量
    xdebug.collect_params=on
    
    ;允许收集函数调用的返回值
    xdebug.collect_return=on
    
    ;启用代码自动跟踪
    xdebug.auto_trace=on
    
    ;性能优化，本文用不到，选择关闭（不关闭，会以约每分钟几百M的速度产生大量日志文件，用不上一天你的硬盘就哭了）
    xdebug.profiler_enable = Off ;关掉性能检测分析
    
    ;指定性能分析信息文件的名称
    xdebug.profiler_output_name = cachegrind.out.%t.%p
    
    ;远程端口，指phpstorm配置的端口
    xdebug.remote_port=9001
    
    ;指定远程调试的处理协议
    xdebug.remote_handler = "dbgp"
    
    ;是否允许远程终端，这个必须开启
    xdebug.remote_enable = on
    ;远程IP地址，就算你phpstorm所在的IP。如果你是在本地的话直接写127.0.0.1就可以了
    xdebug.remote_host=127.0.0.1
    xdebug.idekey = PHPSTORM ;这里是调试器的关键字
    xdebug.remote_autostart=1
    xdebug.remote_mode=req
    
重启`wamp`查看配置是否生效

![img、1580998-20190312105534466-205518377.png](https://i.loli.net/2019/10/29/LNVHcO8fKnDxu1l.png)

#### 2. 在谷歌浏览器中添加`xdebug`插件

![img、03.png](https://i.loli.net/2019/10/29/K2l4mNbFM5BonYI.png)

添加完后的效果如图所示，在插件栏中多了一个小甲虫

![img、1580998-20190312110407321-1371532303.png](https://i.loli.net/2019/10/29/X3dnkbPAIZDRqj4.png)

这时右击小甲虫点击选项，选择`phpstorm` 点击`save`

![img、1580998-20190312110552171-855056154.png](https://i.loli.net/2019/10/29/SNWRsEcBLv4JhH3.png)

#### 3. 配置`phpstorm`
配置本地执行`php.exe`的位置和检查`php`语法的版本




配置`Debug` ：`Languages & Frameworks` -> `PHP` -> `Debug`，只需要把端口改为`9001`，和`xdebug`的配置保持一致

![img、1580998-20190312111325527-1992846745.png](https://i.loli.net/2019/10/29/9LMrI1zjXHqAx4d.png)


.配置`Server`（就在Debug下面一个） ：`Languages & Frameworks` -> `PHP` -> `Servers`，新建一台本地服务器（绿色加号），填写服务器名字以及`host`，确认`debugger`是`xdebug`

![img、1580998-20190312112042013-289121517.png](https://i.loli.net/2019/10/29/n1yteCR4SVaXcBf.png)


启动`xdebug helper`：点击`xdebug helper`图标，选择`Debug`项，灰色图标变成绿色

![img、1580998-20190312112258403-1783506273.png](https://i.loli.net/2019/10/29/yKG6nELFwfRmVOr.png)


在`phpstorm`中将需要调试的代码打上断点，点击右上角电话图标开启调试监听，由一头绿一头红变成两头绿即可

![img、1580998-20190312112351263-1881895127.png](https://i.loli.net/2019/10/29/bU7kSRq1flCg6Av.png)

#### 4. 开始`debug`

在谷歌浏览器中输入配置好的`Url`,会出现如下图所示，恭喜你`phpstorm`和`xdebug`的配置基本完成,这时候就可以愉快的进行调试了

![img、1580998-20190312112749516-731577817.png](https://i.loli.net/2019/10/29/DGg3os8x5MVlTX9.png)



#### 5. 参考文章

[phpStorm+xdebug断点调试环境配置最简实践](https://segmentfault.com/a/1190000010098789)

[如何愉快的在PhpStorm中进行Xdebug断点调试？](https://segmentfault.com/a/1190000014942730)