---
title: 解决ie低版本不认识html5标签
categories: [前端,html5]
tags: [html] 
date: 2019-10-14 20:21:35
---

# 解决ie低版本不认识html5,css3标签

> 在不支持`HTML5`新标签的浏览器里，会将这些新的标签解析成行内元素(`inline`)对待，所以我们只需要将其转换成块元素(`block`)即可使用，但是在`IE9`版本以下，并不能正常解析这些新标签，但是却可以识别通过`document.createElement('tagName')`创建的自定义标签，于是我们的解决方案就是将`HTML5`的新标签全部通过`document.createElement`('tagName')来创建一遍，这样`IE`低版本也能正常解析`HTML5`新标签了。

####  处理方式：在实际开发中我们更多采用的是通过检测IE浏览器的版本来加载三方的一个JS库来解决兼容问题（测试在IE下面的兼容性：ieTester软件的使用）

>`html5shiv`：解决ie9以下浏览器对html5新增标签的不识别，并导致CSS不起作用的问题。<br>
>`respond`:让不支持css3 Media Query的浏览器包括IE6-IE8等其他浏览器支持查询。

 我们解决的问题， 主要是针对于`ie`低版本的，也就是只有低版本`ie`才执行才对。
 
     <!--[if lt IE 9]>  
     　　<script src="//cdn.bootcss.com/respond.js/1.4.2/respond.js"></script>
      　　<script src="http://cdn.bootcss.com/html5shiv/3.7.2/html5shiv.min.js"></script> 
     <![endif]—>

#### 条件注释 

    <!--[if !IE]><!--> 除IE外都可识别 <!--<![endif]-->
    <!--[if IE]> 所有的IE可识别 <![endif]-->
    <!--[if IE 6]> 仅IE6可识别 <![endif]-->
    <!--[if lte IE 6]> IE6以及IE6以下版本可识别 <![endif]-->
    <!--[if gte IE 6]> IE6以及IE6以上版本可识别 <![endif]-->
    <!--[if IE 7]> 仅IE7可识别 <![endif]-->
    <!--[if lt IE 7]> IE7以下版本可识别 <![endif]-->
    <!--[if gt IE 7]> IE7以上版本可识别 <![endif]-->
    <!--[if IE 8]> 仅IE8可识别 <![endif]-->
    <!--[if IE 9]> 仅IE9可识别 <![endif]-->
    
   
![](https://raw.githubusercontent.com/qnyt1993/picture/master/img/2019/09/24/html.jpg)
