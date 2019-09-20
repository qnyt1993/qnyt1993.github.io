---
title: Nginx 配置二级虚拟目录访问 Laravel 重写
date: 2019-09-20 10:30:39
categories: [后端,php,laravel]
tags: [laravel,php,nginx] 
---


    server {
        listen          80;
        server_name    _;
        root            /opt/sites;
        index           index.php index.html index.htm;
        etag on;
    
        gzip on;
        gzip_vary on;
        gzip_http_version 1.0;
        gzip_min_length 1k;
        gzip_buffers 4 16k;
        gzip_comp_level 2;
        gzip_disable msie6;
        gzip_types text/plain text/css application/json application/javascript application/x-javascript text/javascript text/xml application/xml application/xml+rss;
    
        client_max_body_size 110m;
        client_body_buffer_size 1024k;
    
        keepalive_timeout   60;
    
        sendfile on;
        sendfile_max_chunk 512k;
        tcp_nopush on;
        tcp_nodelay on;
    
    
    # 此处配置二级目录站点
       location  /son{
            alias   /opt/sites/cms/public;
            index index.html index.php;
    
            try_files $uri $uri/ /index.php?$query_string;
    
            location ~ \.php$ {
                    if (!-f $request_filename) {
                            return 404;
                    }
    
                    fastcgi_pass        unix:/tmp/php-fpm-72.sock;
                    fastcgi_index       index.php;
                    fastcgi_param       SCRIPT_FILENAME $request_filename;
                    include             fastcgi_params;
            }
        }
    
      #此处是根目录下配置laravel站点
        location / {
            try_files $uri $uri/ /index.php?$query_string;
        }
    
        location ~ \.php$ {
            fastcgi_pass        unix:/tmp/php-fpm-72.sock;
            fastcgi_index       index.php;
            fastcgi_param       SCRIPT_FILENAME $document_root$fastcgi_script_name;
            include             fastcgi_params;
        }
    
        location = /robots.txt  { access_log off; log_not_found off; }
        location = /favicon.ico { access_log off; log_not_found off; }
    }
    
    
这次遇到的问题是在访问`http://localhost/son`访问不了`index.php`，而`index.html`可以访问。在参考了`https://laravel-china.org/articles/14235/nginx-configuring-two-level-virtual-directory-access-laravel-rewrite`解决.