---
title: layer弹出框的简易封装和使用
date: 2019-09-20 15:37:10
categories: [前端,插件]
tags: [弹出框,layer] 
---


#### 1. 封装`layer`
下载`layer`绿色版和`jquery`引入页面

    <!DOCTYPE html>
    <html lang="zh-CN">
    .
    .
    .
    <script src="/Public/js/jquery.js"></script>
    <script src="/Public/js/dialog/layer.js"></script>
    <script src="/Public/js/dialog.js"></script>
    </body>
    </html>
    
#### 2. 封装`dialog.js`

    var dialog = {
        // 错误弹出层
        error: function(message) {
            layer.open({
                content:message,
                icon:2,
                title : '错误提示',
            });
        },
    
        //成功弹出层
        success : function(message,url) {
            layer.open({
                content : message,
                icon : 1,
                yes : function(){
                    location.href=url;
                },
            });
        },
    
        // 确认弹出层
        confirm : function(message, url) {
            layer.open({
                content : message,
                icon:3,
                btn : ['是','否'],
                yes : function(){
                    location.href=url;
                },
            });
        },
    
        //无需跳转到指定页面的确认弹出层
        toconfirm : function(message) {
            layer.open({
                content : message,
                icon:3,
                btn : ['确定'],
            });
        },
    }
    
使用的时候在`js`中直接`dialog.error("提示信息")`即可实现`layer`的效果