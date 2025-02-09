---
title: mysql导入导出
top: false
cover: false
toc: true
mathjax: true
date: 2023-01-26 11:03:04
password: 5c4aaf271c7c13b2372e84982f82f2cd279026950438dfc769c95dcabdfd2a87
summary: mysql数据导入导出
tags: mysql
categories: db
---
## 导出数据库
```shell
mysqldump -u userName -p password 数据库名字 > 位置+文件名字(sql文件)
    
1.导出整个数据库
mysqldump -u 用户名 -p密码 数据库名 > 位置+导出的文件名(sql文件)


2.导出一个表
mysqldump -u 用户名 -p密码 数据库名 表名> 位置+导出的文件名(sql文件)
```

## 导入数据
```shell
# 先创建数据库
set names utf8; # 设置编码
source 文件位置(sql文件)
```