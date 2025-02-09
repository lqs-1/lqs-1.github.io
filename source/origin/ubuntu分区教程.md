---
title: ubuntu分区教程
top: false
cover: false
toc: true
mathjax: true
date: 2023-05-05 14:05:46
password:
summary: ubuntu分区教程
tags: os
categories: ubuntu
---



## ubuntu桌面版安装分区教程

### 制作ubuntu启动盘

1.去官网[`https://ubuntu.com/#download`](https://ubuntu.com/#download)下载`Ubuntu Desktop`的最新版本

2.去ventoy官网[`https://www.ventoy.net/cn/download.html`](https://www.ventoy.net/cn/download.html)下载启动盘制作工具

3.下载好ventoy之后，插上U盘，解压ventoy，双击Ventoy2Disk.ext将U盘制作程启动盘

4.将第一步下载好的ISO镜像文件复制到制作好的U盘中

5.重启系统，狂按你电脑进入Boot Manager的按键，不同品牌的按键有所不同

6.选择你的U盘，再选择你要安装的系统



### 分区

> efi 逻辑分区 512MB-1024MB

```text
因为是u盘的uefi启动，因此设置一个efi引导
```



> swap 主分区 交换空间 大小为物理内存的1.5倍左右

```text
如果物理内存在8G以下，则swap设置为物理内存一样的大小，如果超过8G，则一般设置为8G大小的虚拟内存就足够了。根据自身的使用需求，也可以适当增大swap大小
```



> / 逻辑分区 随便分

```text
主要用来存放ubuntu系统文件。有固态硬盘的可以安在固态盘中
```



>  /home 逻辑分区 随便分

```text
存用户程序，一般在/usr/bin中存放发行版提供的程序，用户自行安装的程序默认安装到/usr/local/bin
```



> /usr 逻辑分区 随便分

```text
存放用户文件。这个分区尽量设置大一些，因为我安装了机械盘，因此分配了300g的存储空间给它
```

