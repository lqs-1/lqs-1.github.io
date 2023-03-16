---
title: minio集群搭建
top: false
cover: false
toc: true
mathjax: true
date: 2023-03-16 12:44:49
password: 5c4aaf271c7c13b2372e84982f82f2cd279026950438dfc769c95dcabdfd2a87
summary: minio集群搭建
tags: minio
categories: storage
---

# 概述

> MinIO 是在 GNU Affero 通用公共许可证 v3.0 下发布的高性能对象存储。它与 Amazon S3 云存储服务 API 兼容。使用 MinIO 为机器学习、分析和应用程序数据工作负载构建高性能基础架构

[`官网`](https://min.io/)

[`github`](https://github.com/minio/minio)

## 基本概念

> - `S3`: Simple Storage Service，简单存储服务，这个概念是Amazon在2006年推出的，对象存储就是从那个时候诞生的。S3提供了一个简单Web服务接口，可用于随时在Web上的任何位置存储和检索任何数量的数据 

> - `Object`: 存储到 Minio 的基本对象，如文件、字节流，Anything

> - `Bucket`: 用来存储 Object 的逻辑空间。每个 Bucket 之间的数据是相互隔离的

> - `Drive`: 部署 Minio 时设置的磁盘，Minio 中所有的对象数据都会存储在 Drive 里

> - `Set`: 一组 Drive 的集合，分布式部署根据集群规模自动划分一个或多个 Set ，每个 Set 中的 Drive 分布在不同位置

## 纠删码（Erasure Code）

> 纠删码（Erasure Code）简称EC，是一种数据保护方法，它将数据分割成片段，把冗余数据块扩展、编码，并将其存储在不同的位置，比如磁盘、存储节点或者其它地理位置


# 集群

> 两台主机，每台主机4个磁盘

## 每台机子创建相关目录
```shell
mkdir /home/minio/{app,run,logs,config,data}
```
`data`:磁盘目录都放这里
`app`:minio可执行文件
`run`:启动脚本
`logs`:日志目录
`config`:配置目录

## 下载minio

> `cd /home/minio/app`

```shell
wget https://dl.min.io/server/minio/release/linux-amd64/minio
chmod +x minio
```

## 每台机子创建四个目录（用于分区挂载）
```shell
mkdir /home/minio/data/{data1,data2,data3,data4} -pv
```

## 分区挂载(每台机子)

查看新磁盘,一般会看到磁盘名称`/dev/sdb`
```shell
fdisk -l
```

对磁盘进行分区
```shell
fdisk /dev/sdb
```

会有提示输入参数
```shell
command (m for help):n # 分区
Select (default p):p # 主分区
Partition number(1-4):enter # 默认就可以只能4个主分区
First cylinder (1-15908,default 1):enter # 默认
Last sector, +sectors or +size{K,M,G,T,P}(2048-10485759， default 10485759):+2G # 分区大小,磁盘大小必须>1G
command (m for help):w # 保存
```

格式化磁盘
```shell
mkfs.ext4 /dev/sdb1
mkfs.ext4 /dev/sdb2
mkfs.ext4 /dev/sdb3
mkfs.ext4 /dev/sdb4
```

挂载
```shell
mount /dev/sdb1 /home/minio/data/data1
mount /dev/sdb2 /home/minio/data/data2
mount /dev/sdb3 /home/minio/data/data3
mount /dev/sdb4 /home/minio/data/data4
```

## 配置

> Minio默认9000端口，在配置文件中加入–address “:9029” 可更改端口

> - `MINIO_ACCESS_KEY`: 用户名，长度最小是5个字符

> - `MINIO_SECRET_KEY`: 密码，密码不能设置过于简单，不然minio会启动失败，长度最小是8个字符

> - `--config-dir`: 指定集群配置文件目录

> - `--address`: api的端口，默认是9000

> - `--console-address`: web端口，默认随机

### 编写启动脚本

> `vim /home/minio/run/run.sh`

```shell
#!/bin/bash
export MINIO_ROOT_USER=admin
export MINIO_ROOT_PASSWORD=admin123456

# 在每台机器上都执行该文件，即以分布式的方式启动了MINIO
# --address ":9000" 挂载9001端口为api端口（如Java客户端）访问的端口
# --console-address ":9000" 挂载9000端口为web端口； 
/home/minio/app/minio server --address ":9000" --console-address ":9001" --config-dir /home/minio/config \
http://local-168-182-110/home/minio/data/data1 \
http://local-168-182-110/home/minio/data/data2 \
http://local-168-182-110/home/minio/data/data3 \
http://local-168-182-110/home/minio/data/data4 \

http://local-168-182-111/home/minio/data/data1 \
http://local-168-182-111/home/minio/data/data2 \
http://local-168-182-111/home/minio/data/data3 \
http://local-168-182-111/home/minio/data/data4 > /home/minio/logs/minio_server.log
```

> 下面脚本复制时 \ 后不要有空格，还有就是上面的目录是对应的一块磁盘，而非简单的在/home/minio/data目录下创建四个目录，要不然会报如下错误，看提示以为是root权限问题`part of root disk, will not be used (*errors.errorString)`

### 启动服务
```shell
# 在每台机器上都执行该文件，即以分布式的方式启动了MINIO
sh /home/minio/run/run.sh
```

#### 添加或修改minio.service，通过systemctl启停服务（推荐）

> `vim /etc/systemd/system/minio.service`

```shell
[Unit]
Description=Minio service
Documentation=https://docs.minio.io/

[Service]
WorkingDirectory=/home/minio
ExecStart=/home/minio/run/run.sh

Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
```

#### 修改文件权限
```shell
chmod +x /etc/systemd/system/minio.service && chmod +x /home/minio/appt/minio && chmod +x /home/minio/run/run.sh
```

#### 启动集群（每台）
```shell
 #重新加载服务
systemctl daemon-reload
#启动服务
systemctl start minio
#加入自启动
systemctl enable minio
```