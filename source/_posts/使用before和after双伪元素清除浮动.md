---
title: 使用before和after双伪元素清除浮动
tags: []
categories: [前端,知识点]
date: 2019-09-24 23:09:50
---

使用方法：


    .clearfix:before,.clearfix:after { 
      content:".";
      display:table;
    }
    .clearfix:after {
     clear:both;
    }
    .clearfix {
      *zoom:1;
    }


优点：  代码更简洁

缺点：  由于IE6-7不支持:after，使用 zoom:1触发 hasLayout。

代表网站： 小米、腾讯等