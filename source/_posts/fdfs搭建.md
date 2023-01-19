---
title: fdfs搭建
top: false
cover: false
toc: true
mathjax: true
date: 2023-01-19 16:14:57
password:
summary: FastDfs搭建
tags: fdfs
categories: 工具
---
## FastDFS
---
### FastDFS安装
#### 安装fastdfs依赖包libfastcommon-master
```shell
./make.sh
sudo ./make.sh install
```
#### 安装fastdfs
```
fastdfs-master目录中
./make.sh
sudo ./make.sh install
```	
#### 配置tracker

```		
cp/etc/fdfs/tracker.conf.sample/etc/fdfs/tracker.conf

在/home/python/目录中创建目录 fastdfs/tracker      			
    mkdir –p /home/python/fastdfs/tracker

编辑/etc/fdfs/tracker.conf配置文件    
    sudo vim /etc/fdfs/tracker.conf
    修改 base_path=xxx
```	
#### 配置storage
```
sudo cp /etc/fdfs/storage.conf.sample /etc/fdfs/storage.conf

在/home/python/fastdfs/ 目录中创建目录 storage
    mkdir –p /home/python/fastdfs/storage

编辑/etc/fdfs/storage.conf配置文件  
    sudo vim /etc/fdfs/storage.conf

修改内容：
    base_path=/home/python/fastdfs/storage
    store_path0=/home/python/fastdfs/storage
    tracker_server=自己ubuntu虚拟机的ip地址:22122
```
#### 配置client

```
base_path=/home/python/fastdfs/tracker
tracker_server=自己ubuntu虚拟机的ip地址:22122
```	

### 上传文件测试

```
fdfs_upload_file /etc/fdfs/client.conf 要上传的图片文件 
```