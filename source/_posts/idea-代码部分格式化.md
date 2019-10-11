---
title: idea 代码部分格式化
tags: [other]
categories: [杂记]
date: 2019-10-11 09:09:53
---

>效果: 处理`Idea`使用`ctrl+alt+L`进行代码格式化时部分代码可以被忽略，不执行格式化功能(`webstorm`,`phpstorm`同理)
>原因: 有时希望自己写的一些代码不被格式化，或者发现格式化后代码显示样式异常

##  配置`idea`开启

`File`-->`Settings`-->`Editor`-->`Code Style` 在代码格式化选项中开启部分代码忽略功能

![](https://raw.githubusercontent.com/qnyt1993/picture/master/img/2019/09/24/QQ%E6%88%AA%E5%9B%BE20191011091945.png)

勾选开启代码部分格式化
点击应用,结束配置

## 对需要忽略格式化的代码进行配置

这里以`css`代码为例

    /*@formatter:off*/
        这里是需要忽略格式化的css代码
    /*@formatter:on*/
    
## 测试效果

源代码

![](https://raw.githubusercontent.com/qnyt1993/picture/master/img/2019/09/24/QQ%E6%88%AA%E5%9B%BE20191011092301.png)


格式化代码

![](https://raw.githubusercontent.com/qnyt1993/picture/master/img/2019/09/24/QQ%E6%88%AA%E5%9B%BE20191011092250.png)


添加代码忽略后，执行格式化后的代码

![](https://raw.githubusercontent.com/qnyt1993/picture/master/img/2019/09/24/QQ%E6%88%AA%E5%9B%BE20191011092223.png)

## 参考

[IDEA(AS)代码格式化部分忽略](https://blog.csdn.net/Mislead/article/details/52130536)

[idea代码部分格式化](https://www.jianshu.com/p/ffc44c50f688)

    