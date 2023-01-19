---
title: nginx搭建
top: false
cover: false
toc: true
mathjax: true
date: 2023-01-19 16:32:38
password:
summary: nginx搭建
tags: nginx
categories: 工具
---
# Nginx

## 各种方式安装的nginx的启动方式
### 通过命令行安装
```shell
apt install nginx
/etc/init.d/nginx start     # 启动
/etc/init.d/nginx stop      # 停止
/etc/init.d/nginx restart   # 重启
```
### 通过docker安装
```
docker start nginx   # 启动
docker stop nginx	 # 停止
docker restart nginx # 重启
```
### 通过编译安装
```
/xx/xx/nginx/sbin/nginx # 启动
/xx/xx/nginx/sbin/nginx -s reload # 重启
ps -ef | grep nginx && kill xxxx # 关闭
kill -9 nginx # 强制关闭
```

## nginx概念
Nginx 是一个高性能的HTTP和反向代理web服务器，同时也提供了IMAP/POP3/SMTP服务

### Nginx特色
```
1.反向代理:
正向代理：客户端配置代理服务器，通过代理服务器进行互联网访问
反向代理：客户端将请求发送给反向代理服务器，由反向代理服务器去选择目标服务器获取数据后，再返回给客户端，此时反向代理服务器和目标服务器对外来说就是一个服务器，暴露得是反向代理服务器的地址，隐藏了真正的服务器地址
2、负载均衡（并发性能提升）：
在客户端发送多次请求的时候，将请求分发到不同的服务器上
3、动静分离：
为了提升网站的解析速度，可以将静态页面和动态页面由不同的服务器来解析，加快解析速度，降低原本单个服务器的压力
```

### nginx的安装

```
nginx:安装的本体（nginx.org）
依赖：
openssl:支持nginx开通https的模块，非对称加密的工具（选择性下载）(https://www.openssl.org/）
zlib:是一款支持zlib压缩的模块(zlib.net)(必须下载)
pcre:是一个支持正则表达式的模块（www.pcre.org）（必须下载）

都下载好以后就在usr/share/下面创建一个nginx文件夹
将下载好的文件解压到nginx文件夹下

安装C语言编译工具：
apt install gcc
apt install build-essential
apt install make

进入解压好的nginx里面：
./configure --with-http_ssl_module --with-openssl=../opensslx-x-x  --with-pcre=../pcrex-x-x --with-zlib=../zlibx-x-x

这个命令可以将模块加载进入nginx

然后用make命令编译一下
make install安装一下

安装在/usr/local/nginx/下面

进入这个目录
启动：
    进入sbin执行./nginx
重启：
    ./nginx -s reload
关闭：
    pkill -9 nginx强制关闭
    ./nginx -s stop
配置：
    进入conf，vim编辑nginx.conf配置nginx
    
```
            
            
## 基本使用
### 配置文件：全局块，events块，http块

```

worker_processes  1; # 线程数


events {
    worker_connections  1024; #最大连接数
}


http {
    include       mime.types;
    default_type  application/octet-stream;
    sendfile        on;
    keepalive_timeout  65;

    upstream xxx {
        # ip_hash # 每个请求按访问ip的hash结果分配，这样每个访客固定访问一个后端服务器，可以解决session的问题。

        server 10.0.6.108:7080 weight=2; # 权重越大被访问的次数就越多，默认是1，轮询 
        server 10.0.0.85:8980 weight=4; 

        # fair # 按后端服务器的响应时间来分配请求，响应时间短的优先分配。与weight分配策略类似。
        # 这三种决策使用其中之一就可以  
        # 这个用于实现负载均衡
        # 在location使用：proxy_pass http://xxx就可以
    }



    server {
    listen       80;
    server_name  localhost;

    # location的一些修饰符：
    # = 用于精确匹配
    # ~ 用于区分大小写的正则匹配，模糊匹配
    # ~* 用于不区分大小写的正则匹配，模糊匹配
    # ^~ 用于字符串前缀匹配，找到最长匹配（高位匹配最多的，做完美的），然后停止查找

    # ^~的优先级大于正则


    location ~ /group[0-9]/ {
            ngx_fastdfs_module;
        }

    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
            root   html;
        }
    }


    # server在http中可以有多个
    # location在server中也可以有多个，用于解决动静分离的时候可以用
}
```

#### 反向代理 
通过proxy_pass 地址