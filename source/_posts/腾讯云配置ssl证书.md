---
title: 腾讯云配置ssl证书
date: 2021-05-10 15:36:11
categories: 博客教程
tags:
- nginx
- https
# sticky: 100
pic:
comments: true
toc: true
only:
- home
- category
- tag
---

## 什么是SSL证书？

SSL证书是用于在Web服务器与浏览器以及客户端之间建立加密链接的加密技术，通过配置和应用SSL证书来启用HTTPS协议，来保证互联网数据传输的安全，全球每天有数以亿计的网站都是通过HTTPS来确保数据安全，保护用户隐私。

## 申请腾讯云SSL证书

百毒搜索腾讯SSL证书，找到免费使用SSL一年的产品，一系列骚操作后得到证书。

## nginx配置修改

下载SSL证书，上传到服务器`/etc/pki/nginx/`目录下。

修改`/etc/`配置文件，注意SSL证书的路径和实际上传的路径和名称一致。

```shell
    # server {
    #     listen       80 default_server;
    #     listen       [::]:80 default_server;
    #     server_name  www.holychan.ltd;
    #     root         /home/hexo;

    #     # Load configuration files for the default server block.
    #     include /etc/nginx/default.d/*.conf;

    #     location / {
    #     }

    #     error_page 404 /404.html;
    #         location = /40x.html {
    #     }

    #     error_page 500 502 503 504 /50x.html;
    #         location = /50x.html {
    #     }
    # }
    # 以上的配置注释，使用下面的配置

    # Settings for a TLS enabled server.

    server {
        listen       443 ssl http2 default_server;
        listen       [::]:443 ssl http2 default_server;
        server_name  www.holychan.ltd;
        root         /home/hexo;    # 根目录

        ssl_certificate "/etc/pki/nginx/1_holychan.ltd_bundle.crt";    # ssl证书路径
        ssl_certificate_key "/etc/pki/nginx/2_holychan.ltd.key"; # ssl证书密钥
        ssl_session_cache shared:SSL:1m;
        ssl_session_timeout  10m;
        ssl_ciphers PROFILE=SYSTEM;
        ssl_prefer_server_ciphers on;

        # Load configuration files for the default server block.
        include /etc/nginx/default.d/*.conf;

        location / {
        }

        error_page 404 /404.html;
            location = /40x.html {
        }

        error_page 500 502 503 504 /50x.html;
            location = /50x.html {
        }
    }

}
```