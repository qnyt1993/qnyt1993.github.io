---
title: 腾讯云免费申请https证书
date: 2019-09-20 12:32:22
categories: [杂记]
tags: [https] 
---

> 首先有一台腾讯云服务器以及一个腾讯云已备案的域名

#### 1. 打开`https://cloud.tencent.com/product/ssl`

#### 2. 点击立即购买

![SSL证书选购页面](https://raw.githubusercontent.com/qnyt1993/picture/master/img/2019/09/20/20190920123826.png)

#### 3. 点击免费快速申请

![证书申请页面](https://raw.githubusercontent.com/qnyt1993/picture/master/img/2019/09/20/20190920124016.png)

#### 4. 根据提示填写(私钥密码可以不填)，点击下一步

![证书申请页面](https://raw.githubusercontent.com/qnyt1993/picture/master/img/2019/09/20/20190920124057.png)


#### 5. 点击确认申请
基本上`Https`的证书申请流程差不多走完了，这时候只要审核通过即可

#### 6. 配置https证书
将`https`证书下载下来，默认是一个`zip`包，解压打开后有

![](https://raw.githubusercontent.com/qnyt1993/picture/master/img/2019/09/20/20190920124202.png)

#### 7. 根据自己的服务器选择对应的证书,我这里用的是`nginx`,将证书两个上传到对应的腾讯云服务器,要给予证书执行权限

![](https://raw.githubusercontent.com/qnyt1993/picture/master/img/2019/09/20/20190920124228.png)

#### 8. 在`nginx`的配置文件中添加`https`的配置

    cd /etc/nginx/conf.d
    vim 443.conf
    
填入如下配置

    server {
        listen       443;
        server_name  127.0.0.1;
        ssl on;
        ssl_certificate /etc/nginx/ssl/server.crt;
        ssl_certificate_key /etc/nginx/ssl/server.key;
    
        ssl_session_timeout 5m;
        ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:ECDHE:ECDH:AES:HIGH:!NULL:!aNULL:!MD5:!ADH:!RC4;
        ssl_protocols TLSv1 TLSv1.1 TLSv1.2;        
        ssl_prefer_server_ciphers on;
        ssl_verify_depth 2;
        root /opt/sites ;
     
        location / {
            index  index.html index.htm;
        }
    
        location = /50x.html {
            root   /usr/share/nginx/html;
        }
    
        location ~ \.php$ {
            fastcgi_pass   127.0.0.1:9000;
            fastcgi_index  index.php;
            fastcgi_param  SCRIPT_FILENAME  $document_root$fastcgi_script_name;
            include        fastcgi_params;
        }
    
    }

保存退出

#### 9. 重启nginx

    nginx -s reload
    