---
title: Linux环境变量配置
top: false
cover: false
toc: true
mathjax: true
date: 2023-05-10 06:42:53
password: 5c4aaf271c7c13b2372e84982f82f2cd279026950438dfc769c95dcabdfd2a87
summary: Linux环境变量配置
tags: 环境变量
categories: Linux
---

### Linux配置环境变量

> 以`/etc/profile`为例

```shell
# 只配置一个环境变量如java export JAVA_HOME=xxxxxxxxxxxxxxxx # 添加一个变量 路径 export PATH=$PATH:$JAVA_HOME/bin # 给PATH添加一个变量,$表示引用,这里可以使用配好的路径变量也可以直接写路径 # 配置多个环境变量 export JAVA_HOME=xxxxxxxxxxxxxxxx # 添加一个变量 路径 export REDIS_HOME=xxxxxxxxxxxxxxxx # 添加一个变量 路径 export MAVEN_HOME=xxxxxxxxxxxxxxxx # 添加一个变量 路径 export NODE_HOME=xxxxxxxxxxxxxxxx # 添加一个变量 路径 export PATH=$PATH:$JAVA_HOME/bin:$REDIS_HOME/bin:$MAVEN_HOME/bin:$NODE_HOME/bin # 配置多个环境变量的时候可以用`:`分割,放在同一行
```
