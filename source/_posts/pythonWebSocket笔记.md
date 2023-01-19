---
title: pythonWebSocket笔记
top: false
cover: false
toc: true
mathjax: true
date: 2023-01-19 15:04:40
password:
summary: webSocket
tags: basic
categories: Python
---
## udp的数据通信
```
1、socket:创建套接字
2、bind:绑定本地端口，接收数据必须绑定，发送数据可以不绑定
3、sendto、recvfrom:接收发送数据
4、close:关闭套接字
```
## tcp的数据通信
>    客户端
```
1、socket:创建套接字
*、bind:绑定端口号，客户端可以不用绑定，使用随机端口即可
2、connect:链接服务器
3、send、recv:接收发送数据
4、close:关闭套接字
```

>   服务器
```text
1、socket:创建套接字
*、setsock：套接字.setsockopt(SOL_SOCKET, SO_REUSEADDR, 1)，在客户端关闭之前，服务器首先关闭，关闭（close)
2、bind:一定要绑定端口信息
3、listen:使得主动变成被动
4、accept:等待客户端链接
5、send、recv:收发数据
6、close:关闭套接字
```