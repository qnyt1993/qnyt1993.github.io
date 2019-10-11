---
title: Fedora添加软件桌面快捷方式
categories: [运维,fedora]
tags: [fedora] 
date: 2019-10-11 12:28:47
---

## 以下以添加`Eclipse`为例

在桌面上新建`Eclipse.desktop` 文件，向其写入如下代码

    [Desktop Entry]  
    Name=Eclipse  
    Comment=用Eclipse开发  
    Exec=/usr/lib/eclispe/eclipse  
    Icon=/usr/lib/eclipse/eclipse32.png 
    Terminal=false 
    Type=Application 
    Categories=Application;Development;

注意修改 `Exec`软件执行路径以及`Icon`软件icon的路径

将其标记为`信任`：

直接点击它，会提示标记为`信任`。

或者右击选择`属性`，在`权限选项卡`中勾选`允许程序执行文件`

下面将其放入应用程序中：

打开终端，将其拷贝到 `/usr/share/applications/`目录下（需要`root`权限）

## 参考

[如何在Fedora添加桌面快捷方式、如何添加到应用程序](https://blog.csdn.net/linuxchyu/article/details/16984683?utm_source=copy)