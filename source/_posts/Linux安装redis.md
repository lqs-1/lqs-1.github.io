---
title: Linux安装redis
top: false
cover: false
toc: true
mathjax: true
date: 2023-05-10 06:35:04
password: 5c4aaf271c7c13b2372e84982f82f2cd279026950438dfc769c95dcabdfd2a87
summary: Linux Install Redis
tags: redis
categories: Linux
---

## Linux Install Redis
### GET REDIS
> 下载稳定版Redis

```shell
wget https://download.redis.io/redis-stable.tar.gz
```

> 解压Redis到`/usr/local/redis/`

```shell
mkdir /usr/local/redis/ && tar -zxvf redis-stable.tar.gz -C /usr/local/redis/
```

### 安装redis

```shell
mkdir /usr/local/redis/ && tar -zxvf redis-stable.tar.gz -C /usr/local/redis/
```

> 编译redis

```shell
make distclean && make # 清除之前的编译缓存再次编译
```

> 安装

```shell
make PREFIX=/usr/local/redis install # PREFIX不指定的话默认将命令安装到/usr/local/bin
```

> 配置环境变量

```shell
export PATH=$PATH:/usr/local/redis/bin
```

> 复制配置文件

```shell
cp /usr/local/redis/redis-stable/redis.conf /etc/redis
```

