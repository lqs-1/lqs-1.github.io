---
title: Windows安装nginx
top: false
cover: false
toc: true
mathjax: true
date: 2023-05-19 13:55:00
password: 5c4aaf271c7c13b2372e84982f82f2cd279026950438dfc769c95dcabdfd2a87
summary: Windows安装nginx
tags: nginx
categories: windows
---
## Windows Install Nginx

### 下载nginx
> [`下载Windows版的稳定版nginx`](http://nobibibi.top/somg/file/nginx/windows/nginx-1.24.0.zip)并且解压

### 启动nginx
> - 方式一：直接双击`nginx.exe`，双击之后黑窗口一闪而过

> - 方式二：在`nginx.exe`目录下打开cmd，输入`start nginx`

> 查看端口是否被占用`netstat -ano | findstr "80"`

> 重启在cmd中`nginx -s reload`

> 关闭服务`nginx -s reload`，在修改配置以后可以使用
