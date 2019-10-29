---
title: window下的php命令行输出汉字乱码处理
tags: [php]
categories: [杂记]
date: 2019-10-11 11:15:10
---

## 1. 在`php`的代码中加入

    header("content-type:text/html;charset=gbk");
    
## 2. 设置命令行的字体
在命令行上右击`属性` 字体 选择如下字体 点击`确定`


![QQ截图20191029185843.png](https://i.loli.net/2019/10/29/psCeirqRdYo4HFz.png)

我按照上面的流程基本解决问题，如果还有乱码的话，看看是不是编码不是`gbk`尝试`gb2312`以及其他的一些编码。