---
title: mysql主从复制
top: false
cover: false
toc: true
mathjax: true
date: 2023-05-22 18:55:05
password: 5c4aaf271c7c13b2372e84982f82f2cd279026950438dfc769c95dcabdfd2a87
summary: mysql主从复制
tags: mysql
categories: db
---

## 单节点问题
如果数据库是单节点，那么这台数据库如果宕机，那么就不能正常使用应用，还有可能丢失数据

## mysql主从复制

### 主节点配置

> 在主节点的配置文件`my.cnf`中的`[mysqld]`块中修改或者添加配置
```shell
server-id=1    # 服务器 id，随意，但要唯一
log-bin=mysql-bin   # 二进制文件存放路径
binlog-do-db=scada    # 待同步的数据库日志
binlog-ignore-db=mysql  # 不同步的数据库日志
```

> 创建一个用户，特意为了主从复制使用
```shell
# 创建用户 我这里用户名为copyuser，注意这里的ip是从库服务器的ip
CREATE USER 'alave'@'%' IDENTIFIED WITH mysql_native_password BY 'alave';
# 给主从复制账号授权
grant replication slave on *.* to 'alave'@'%';
# 刷新权限
FLUSH PRIVILEGES;
```

> 重启mysql
```shell
service mysql restart
```

> 登录mysql查看master信息
```shell
show master status; # 查看File和Position，在配置从节点的时候会使用
```

### 配置从节点
> 在从节点的配置文件`my.cnf`中的`[mysqld]`块中修改或者添加配置
```shell
server-id=2    # 服务器 id，随意，但要唯一
log-bin=mysql-bin   # 二进制文件存放路径
replicate-do-db=scada    # 待同步的数据库
replicate-ignore-db=mysql,information_schema,performance_schema  # 不同步的数据库
```
> 重启mysql
```shell
service mysqld restart
```

> 登录mysql具体配置如下

```shell
# 登录mysql，然后执行后续代码
mysql -u root -p密码
# 关闭从库
stop slave;
# 设置同步，注意这里是主库ip，日志名称和位置是我们之前上图中看到的名称和位置
change master to master_host='192.xx.xx.111',master_user='slave',master_password='slave',master_log_file='mysql-bin.000002',master_log_pos=xxx;
# 开启从库
start slave; 
#  检查服务器状态 正常情况下Slave_SQL_Running_State是Replica has read all relay log;  waiting for more updates
show slave status \G;
```

### 测试
在主节点添加一条数据，再打开从节点对应的表，会发现已经同步好了