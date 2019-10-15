---
title: 微信JSSdk实现分享功能
categories: [后端,微信]
tags: [weixin] 
date: 2019-10-15 12:06:21
---

# 微信JSSdk实现分享功能

## 1. 概述

 微信分享服务器的作用是为用户在微信浏览器端对来自网站以及客户端的页面进行二次分享链接时更友好的展示提供服务。为实现二次分享功能需要使用微信JS-SDK来开发.

> 微信`JS-SDK`是[微信公众平台](https://mp.weixin.qq.com/cgi-bin/loginpage?t=wxm2-login&lang=zh_CN) 面向网页开发者提供的基于微信内的网页开发工具包。微信`JS-SDK`功能很多。
> 通过使用微信JS-SDK，网页开发者可借助微信高效地使用拍照、选图、语音、位置等手机系统的能力，同时可以直接使用微信分享、扫一扫、卡券、支付等微信特有的能力，为微信用户提供更优质的网页体验。

## 2. 流程图

![](https://raw.githubusercontent.com/qnyt1993/picture/master/img/2019/10/15/QQ%E6%88%AA%E5%9B%BE20191015103943.png)


## 3. 微信二次分享功能实现的具体步骤

#### **步骤一：绑定域名**(微信公众平台配置)

先登录微信公众平台进入“公众号设置”的“功能设置”里填写“JS接口安全域名”。

备注：登录后可在“开发者中心”查看对应的接口权限。

![](https://raw.githubusercontent.com/qnyt1993/picture/master/img/2019/10/15/20190103210645498.png)

需要配置白名单


#### **步骤二：引入`JS`文件**（需要分享的页面）

在需要调用`JS`接口的页面引入如下`JS`文件，（支持`https`）：http://res.wx.qq.com/open/js/jweixin-1.4.0.js

如需进一步提升服务稳定性，当上述资源不可访问时，可改访问：http://res2.wx.qq.com/open/js/jweixin-1.4.0.js （支持`https`）。


	<script src="http://res.wx.qq.com/open/js/jweixin-1.4.0.js"></script>


#### **步骤三：通过`config`接口注入权限验证配置**(微信分享服务器主要提供下面参数生成)

所有需要使用`JS-SDK`的页面必须先注入配置信息，否则将无法调用


    wx.config({
      debug: true, // 开启调试模式,调用的所有api的返回值会在客户端alert出来，若要查看传入的参数，可以在pc端打开，参数信息会通过log打出，仅在pc端时才会打印。
      appId: '', // 必填，公众号的唯一标识
      timestamp: , // 必填，生成签名的时间戳
      nonceStr: '', // 必填，生成签名的随机串
      signature: '',// 必填，签名
      jsApiList: [] // 必填，需要使用的JS接口列表
    });


签名算法见文末的[附录1](https://developers.weixin.qq.com/doc/offiaccount/OA_Web_Apps/JS-SDK.html#62)，所有`JS`接口列表见文末的[附录2](https://developers.weixin.qq.com/doc/offiaccount/OA_Web_Apps/JS-SDK.html#63)



#### **步骤四：通过ready接口处理成功验证**（待分享的页面配置二次分享的页面信息如标题，缩略图，描述等）


    wx.ready(function(){
      // config信息验证后会执行ready方法，所有接口调用都必须在config接口获得结果之后，config是一个客户端的异步操作，所以如果需要在页面加载时就调用相关接口，则须把相关接口放在ready函数中调用来确保正确执行。对于用户触发时才调用的接口，则可以直接调用，不需要放在ready函数中。
        # 分享到朋友圈按钮点击状态及自定义分享内容
        wx.onMenuShareTimeline({
          title: '', // 分享标题
          link: '', // 分享链接，该链接域名或路径必须与当前页面对应的公众号JS安全域名一致
          imgUrl: '', // 分享图标
          success: function () {
          // 用户点击了分享后执行的回调函数
            }
        });
    
        # 分享到QQ按钮点击状态及自定义分享内容
        wx.onMenuShareQQ({
          title: '', // 分享标题
          desc: '', // 分享描述
          link: '', // 分享链接
          imgUrl: '', // 分享图标
          success: function () {
          // 用户确认分享后执行的回调函数
          },
          cancel: function () {
          // 用户取消分享后执行的回调函数
          }
        });
    
        # 分享到腾讯微博按钮点击状态及自定义分享内容
        wx.onMenuShareWeibo({
          title: '', // 分享标题
          desc: '', // 分享描述
          link: '', // 分享链接
          imgUrl: '', // 分享图标
          success: function () {
          // 用户确认分享后执行的回调函数
          },
          cancel: function () {
          // 用户取消分享后执行的回调函数
          }
        });
    
        # 分享到QQ空间按钮点击状态及自定义分享内容
        wx.onMenuShareQZone({
          title: '', // 分享标题
          desc: '', // 分享描述
          link: '', // 分享链接
          imgUrl: '', // 分享图标
          success: function () {
          // 用户确认分享后执行的回调函数
          },
          cancel: function () {
          // 用户取消分享后执行的回调函数
          }
        });
    
    });


## 4. 如何提供权限验证配置（微信分享服务器的业务功能）

即如下参数


    wx.config({
      debug: true, // 开启调试模式,调用的所有api的返回值会在客户端alert出来，若要查看传入的参数，可以在pc端打开，参数信息会通过log打出，仅在pc端时才会打印。
      appId: '', // 必填，公众号的唯一标识
      timestamp: , // 必填，生成签名的时间戳
      nonceStr: '', // 必填，生成签名的随机串
      signature: '',// 必填，签名
      jsApiList: [] // 必填，需要使用的JS接口列表这里我们使用微信分享相关的js接口列表["checkJsApi","onMenuShareTimeline","onMenuShareAppMessage","onMenuShareQQ","onMenuShareWeibo"] 
    });


下面主要说明`signature`的生成流程

1. 获取`access_token`

2. 使用获取到的`access_token`获取`jsapi_ticket`

3. 将排序后的参数URL键值对的格式（即`key1=value1&key2=value`）拼接成字符串,`sha1`签名，得到`signature`

### 4.1 获取`access_token`

> `access_token`是公众号的全局唯一接口调用凭据，公众号调用各接口时都需使用`access_token`。开发者需要进行妥善保存。`access_token`的存储至少要保留512个字符空间。`access_token`的有效期目前为2个小时，需定时刷新，重复获取将导致上次获取的`access_token`失效。

**接口调用请求说明**


    ## https请求方式: GET
    https://api.weixin.qq.com/cgi-bin/token?grant_type=client_credential&appid=APPID&secret=APPSECRET


**参数说明**

| 参数         | 是否必须 | 说明                                      |
| :----------- | :------- | :---------------------------------------- |
| `grant_type` | 是       | 获取`access_token`填写`client_credential` |
| `appid`      | 是       | 第三方用户唯一凭证                        |
| `secret`     | 是       | 第三方用户唯一凭证密钥，即`appsecret`     |

**返回说明**

正常情况下，微信会返回下述`JSON`数据包给公众号：


    {
        "access_token":"ACCESS_TOKEN",
        "expires_in":7200
    }


**参数说明**

| 参数           | 说明                   |
| :------------- | :--------------------- |
| `access_token` | 获取到的凭证           |
| `expires_in`   | 凭证有效时间，单位：秒 |

### 4.2 获取`jsapi_ticket`

> `jsapi_ticket`是公众号用于调用微信`JS`接口的临时票据。正常情况下，`jsapi_ticket`的有效期为7200秒，通过`access_token`来获取。由于获取`jsapi_ticket`的`api`调用次数非常有限，频繁刷新`jsapi_ticket`会导致`api`调用受限，影响自身业务，开发者必须在自己的服务全局缓存`jsapi_ticket` 。

1. 参考以下文档获取`access_token`（有效期7200秒，开发者必须在自己的服务全局缓存`access_token`）：https://developers.weixin.qq.com/doc/offiaccount/Basic_Information/Get_access_token.html
2. 用第一步拿到的`access_token `采用`http` `GET`方式请求获得`jsapi_ticket`（有效期7200秒，开发者必须在自己的服务全局缓存`jsapi_ticket`）：https://api.weixin.qq.com/cgi-bin/ticket/getticket?access_token=ACCESS_TOKEN&type=jsapi

成功返回如下`JSON`：


    {
      "errcode":0,
      "errmsg":"ok",
      "ticket":"bxLdikRXVbTPdHSM05e5u5sUoXNKd8-41ZO3MhKoyN5OfkWITDGgnr2fwJ0m9E8NYzWKVZvdVtaUgWvsdshFKA",
      "expires_in":7200
    }


获得`jsapi_ticket`之后，就可以生成`JS-SDK`权限验证的签名了。

### 4.3 生成签名

签名生成规则如下：参与签名的字段包括`noncestr`（随机字符串）, 有效的`jsapi_ticket`, `timestamp`（时间戳）, `url`（当前待分享网页URL，不包含#及其后面部分） 。对所有待签名参数按照字段名的ASCII 码从小到大排序（字典序）后，使用URL键值对的格式（即`key1=value1&key2=value2…`）拼接成字符串`string1`。这里需要注意的是所有参数名均为小写字符。对`string1`作`sha1`加密，字段名和字段值都采用原始值，不进行`URL` 转义。

即`signature=sha1(string1)`。 示例：


    noncestr=Wm3WZYTPz0wzccnW
    jsapi_ticket=sM4AOVdWfPE4DxkXGEs8VMCPGGVi4C3VM0P37wVUCFvkVAy_90u5h9nbSlYy3-Sl-HhTdfl2fzFy1AOcHKP7qg
    timestamp=1414587457
    url=当前待分享网页URL


步骤1. 对所有待签名参数按照字段名的`ASCII `码从小到大排序（字典序）后，使用URL键值对的格式（即`key1=value1&key2=value2…`）拼接成字符串`string1`：


    jsapi_ticket=sM4AOVdWfPE4DxkXGEs8VMCPGGVi4C3VM0P37wVUCFvkVAy_90u5h9nbSlYy3-Sl-HhTdfl2fzFy1AOcHKP7qg&noncestr=Wm3WZYTPz0wzccnW&timestamp=1414587457&url=http://mp.weixin.qq.com?params=value


步骤2. 对`string1`进行`sha1`签名，得到`signature`：


	0f9de62fce790f9a083d5c99e95740ceb90c27ed


注意事项

1. 签名用的`noncestr`和`timestamp`必须与`wx.config`中的`nonceStr`和`timestamp`相同。
2. 签名用的`url`必须是调用`JS`接口页面的完整`URL`。
3. 出于安全考虑，开发者必须在服务器端实现签名的逻辑。

### 最终效果如下

![](https://raw.githubusercontent.com/qnyt1993/picture/master/img/2019/10/15/QQ%E6%88%AA%E5%9B%BE20191015091334.png)

