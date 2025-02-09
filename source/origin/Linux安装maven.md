---
title: Linux安装maven
top: false
cover: false
toc: true
mathjax: true
date: 2023-05-10 06:40:09
password: 5c4aaf271c7c13b2372e84982f82f2cd279026950438dfc769c95dcabdfd2a87
summary: Linux Install Maven
tags: maven
categories: Linux
---

## Linux Install Maven

> 获取maven

```shell
# 官方
wget https://dlcdn.apache.org/maven/maven-3/3.9.1/binaries/apache-maven-3.9.1-bin.tar.gz
# 或者
wget http://nobibibi.top/somg/file/maven/apache-maven-linux-3.9.1-bin.tar.gz
```

> 解压到`/usr/local/mvn/`

```shell
mkdir /usr/local/mvn && tar -zxvf apache-maven-3.9.1-bin.tar.gz -C /usr/local/mvn/
```

> 配置环境变量

```shell
export PATH=$PATH:/usr/local/mvn/apache-maven-3.9.1/bin
```