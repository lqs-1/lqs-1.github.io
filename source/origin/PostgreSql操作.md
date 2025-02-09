---
title: PostgreSql操作
top: false
cover: false
toc: true
mathjax: true
date: 2023-03-02 16:57:51
password: 5c4aaf271c7c13b2372e84982f82f2cd279026950438dfc769c95dcabdfd2a87
summary: PostgreSql操作
tags: db
categories: pgsql
---

## PostgreSql

### 创建表空间

创建本地表空间存放目录
```
mkdir -pv /home/pgsql13/postgresql/pgdata/dir1
chown -R postgres:postgres /home/pgsql13/postgresql/pgdata
```

创建表空间
```shell
create tablespace 表空间名 owner postgres location '表空间本地目录';
```

### 创建数据库

```shell
CREATE DATABASE 数据库名字 WITH OWNER = postgres ENCODING = 'UTF8' LC_COLLATE = 'zh_CN.UTF-8' LC_CTYPE = 'zh_CN.UTF-8' TABLESPACE = 表空间名 CONNECTION LIMIT = -1;
```

### 用户角色创建

角色即用户
```shell
CREATE ROLE 角色名 WITH LOGIN SUPERUSER NOCREATEDB CREATEROLE INHERIT NOREPLICATION CONNECTION LIMIT -1 PASSWORD '密码';
```

### 架构模式创建

```shell
# 删除自动生成的默认架构即模式
drop schema public cascade;
# 每行会创建一个架构即模式 一般角色名和架构名字一样
CREATE SCHEMA 架构名 AUTHORIZATION 角色名;
# 每行授权一个架构即模式
GRANT ALL ON SCHEMA 角色名 TO 结构名 WITH GRANT OPTION;
```

### 数据导出

> 备份某个库下面的某个架构的数据包括架构，并指定了用户角色

```shell
pg_dump -f xxx-$(date +"%y%m%d").dmp -p 端口 -U 登录用户名 --no-password -v -cC --role=角色 -Fc -b -E UTF8 -n 架构 -d 数据库名
```

> 备份整个库的数据包括架构

```shell
pg_dump -f xxx-$(date +"%y%m%d").dmp -p 端口 -U 登录用户名 --no-password -v -cC -Fc -b -E UTF8 -d 数据库名
```

> 备份整个库的架构不包括数据

```shell
pg_dump -f xxx-$(date +"%y%m%d").dmp -p 端口 -U 登录用户名 --no-password -s -cC -Fc -b -E UTF8 -d 数据库名
```

> 备份某个架构的所有视图

```shell
 pg_dump -p 端口 -U 登录用户名 --no-password -v -cC -t 架构.view_"*" -t 架构.casesearchproject -d 数据库名 > view_all-$(date +"%y%m%d").sql
```

### 数据导入

恢复某个架构包括数据
```shell
pg_restore -p 端口 -d 数据库名 -O -x -v xxx-$(date +"%y%m%d").dmp --role=角色名 -n 架构名
```

恢复某个架构不包括数据
```shell
pg_restore -p 端口 -d 数据库名 -O -x -v xxx-$(date +"%y%m%d").dmp -s --role=角色名 -n 架构名
```

恢复整个数据库包括数据
```shell
pg_restore -p 端口 -d 数据库名 -O -x -v xxx-$(date +"%y%m%d").dmp
```
恢复整个数据库不包括数据
```shell
pg_restore -p 端口 -d 数据库名 -O -x -s -v xxx-$(date +"%y%m%d").dmp
```

恢复某个架构的所有视图
```shell
psql -p 端口 -d 数据库名 -v -f view_all-$(date +"%y%m%d").sql
```


