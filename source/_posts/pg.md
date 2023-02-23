---
title: pg
top: false
cover: false
toc: true
mathjax: true
date: 2023-02-22 21:45:23
password:
summary: 视图、表的备份和恢复
tags: postgresql
categories: sql
---

## postgresql视图和表的导入导出

看文档的方法，对表和视图都没有用，可能是版本的原因

### 备份数据表或者视图

#### 格式

```shell
pg_dump -p port -U username --no-password -v -cC -b -t schema.view|table -t schema.view|table -T schema.view|table -T schema.view|table -d database > xxx.sql
```

> `-p` 指定端口

> `-U` 指定用户

> `--no-password` 密码，这里表示没有密码

> `-v` 展示细节

> `-cC` 在重新创建之前，先清除

> `-b` 在转储中包含大对象

> `-t` 转存的表或者视图等，可以有多个

> `-T` 转存除了这个指定的表或试图，可以有多个
 
> `-d` 属于哪个数据库

#### 例子

```shell
pg_dump -p 11543 -U postgres --no-password -v -cC -b  -t onis.view_visit -t onis.view_study_all -d coreonisdb > 1.sql
```

```shell
pg_dump -p 11543 -U postgres --no-password -v -cC -b -t onis.view_visit -t onis.view_study_all -d tmonisdatadesc > 1.sql
```

```shell
pg_dump -p 11543 -U postgres --no-password -v -cC -b -t onis.view_study -t onis.view_study_flow -t onis.view_studyrecord -t onis.studyvisit -t onis.view_studyvisit_flow -d tmonisdatadesc > 2.sql
```

```shell
pg_dump -p 11543 -U postgres --no-password -v -cC -b -t onis.view_* -t onis.casesearchproject -T onis.view_visit -T onis.view_study_all -T onis.view_study -T onis.view_study-_flow -T onis.view_studyrecord -T view_studyvisit -T onis.view_studyvisit_flow  -d tmonisdatadesc > 3.sql
```

### 恢复

#### 格式

```shell
psql -p port -d database -f backupfile
```

> `-p` 端口

> `-d` 数据库

> `-f` 要恢复的文件

#### 例子

```shell
psql -p 11543 -d coreonisdb -f 1.sql
```