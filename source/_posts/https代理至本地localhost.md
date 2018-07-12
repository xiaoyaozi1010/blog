---
title: https代理至本地localhost
tags: 
    - https
    - nginx
---

# https代理至本地localhost

场景：小程序内嵌了某线上地址，需要修改业务逻辑与小程序联调。而小程序webview业务url不好修改为localhost或http协议的地址。
所以，需要本地借助nginx起一个与线上域名相同的https服务，代替线上服务。

<!-- more -->

## 工具
- nginx
- openssl
- openssl-devel
- switchhosts

## 步骤

### 1.安装nginx

mac 一键安装

```
brew install nginx
```

### 2.安装openssl
mac 一键安装

```
brew install openssl
```
如果提示openssl已经安装过了，link一下

```
brew link openssl
```

### 3.创建https相关文件
建立一个用于存放ssl相关文件的目录，也可以直接放在nginx目录下

```
cd /usr/local/etc/nginx
```

创建各类ssl文件

```
openssl genrsa -des3 -out server.key 1024
openssl req -new -key server.key -out server.csr
openssl rsa -in server.key -out server_nopwd.key
openssl x509 -req -days 365 -in server.csr -signkey server_nopwd.key -out server.crt
```

### 4.配置nginx.conf
nginx.conf在`/usr/local/etc/nginx/conf`里。

localhost:3000可以代替成你希望的地址。

```
http {

    include      mime.types;

    default_type  application/octet-stream;

    send_timeout 1800;

    sendfile        on;

    keepalive_timeout  6500;

    server {

        listen      80;

        server_name  localhost;

        location / {

          proxy_pass          http://localhost:3000;

          proxy_set_header    Host            $host;

          proxy_set_header    X-Real-IP        $remote_addr;

          proxy_set_header    X-Forwarded-For  $proxy_add_x_forwarded_for;

          proxy_set_header    X-Client-Verify  SUCCESS;

          proxy_set_header    X-Client-DN      $ssl_client_s_dn;

          proxy_set_header    X-SSL-Subject    $ssl_client_s_dn;

          proxy_set_header    X-SSL-Issuer    $ssl_client_i_dn;

          proxy_read_timeout 1800;

          proxy_connect_timeout 1800;

        }

    }

    # HTTPS server


    server {

        listen      443;

        server_name  localhost;


        ssl                  on;

        ssl_certificate      server.crt;

        ssl_certificate_key  server.key;

        ssl_session_timeout  5m;


        ssl_protocols  SSLv2 SSLv3 TLSv1;

        ssl_ciphers  ALL:!ADH:!EXPORT56:RC4+RSA:+HIGH:+MEDIUM:+LOW:+SSLv2:+EXP;

        ssl_prefer_server_ciphers  on;


        location / {

          proxy_pass          http://localhost:3000; # 你希望代理到的本地地址

          proxy_set_header    Host            $host;

          proxy_set_header    X-Real-IP        $remote_addr;

          proxy_set_header    X-Forwarded-For  $proxy_add_x_forwarded_for;

          proxy_set_header    X-Client-Verify  SUCCESS;

          proxy_set_header    X-Client-DN      $ssl_client_s_dn;

          proxy_set_header    X-SSL-Subject    $ssl_client_s_dn;

          proxy_set_header    X-SSL-Issuer    $ssl_client_i_dn;

          proxy_read_timeout 1800;

          proxy_connect_timeout 1800;

        }

    }

}
```

### 5.启动nginx
如果生成ssl文件时存在nginx目录下的话，需要加sudo

```
sudo nginx
```

### 5.绑定hosts

没有switchhosts这个工具的话，手工编辑`/private/etc/hosts`文件也可以。

hosts添加：

```
127.0.0.1 xxx.com
```

其中xxx.com是你要代理的线上域名，不要加协议(https)。此时浏览器内访问`https://xxx.com`就会跳往代理地址了(上面的localhost:3000)。

