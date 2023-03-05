---
title: Postgresql安装部署
top: false
cover: false
toc: true
mathjax: true
date: 2023-03-02 16:15:25
password:
summary: postgresql单机安装部署
tags: db
categories: pgsql
---

## PostgreSql

### 下载postgresql源码

> 以Ubuntu为例

下载PostgreSql源码包
```shell
wget https://ftp.postgresql.org/pub/source/v13.0/postgresql-13.0.tar.gz
```

解压源码包
```shell
tar -zxvf postgresql-13.0.tar.gz -C /home/
```

### 安装前准备工作

安装编译工具及相关依赖
```shell
 apt-get -y install zlib1g-dev libreadline-dev # 安装依赖
 apt-get -y install binutils
 apt-get -y install gcc
 apt-get -y install g++
 apt-get -y install sysstat
 apt-get -y install unixodbc unixodbc-dev
```

创建用户
```shell
adduser postgres
```

postgres用户提权`/etc/sudoers`
```
postgres ALL=(ALL:ALL) ALL
```

创建postgresql相关目录和文件夹所属关系
```shell
mkdir /home/postgres/data
mkdir /home/postgres/init
chown -R postgres:postgres /home/postgres
```

### 编译安装PostgreSql

进入源码目录
```shell
cd /home/postgresql-13.0
```

configure执行配置准备构建环境
```
./configure --with-pgport=5432 \
  --prefix=/usr/local/pgsql \
  --with-systemd \
  --with-blocksize=8 \
  --with-segsize=1 \
  --with-wal-blocksize=8 \
  --without-readline \
  --datadir=/home/postgres/init
```

> configure参数说明

```shell
--with-pgport 指定端口
--prefix  安装目录
--with-systemd 编译对systemd 服务通知的支持。如果服务器是在systemd机制下被启动，这可以提高集成度，否则不会有影响 。要使用这个选项，必须安装libsystemd 以及相关的头文件
--with-blocksize 设置块大小，单位K。2的N次幂，1-32K之间
--with-segsize 超大数据表文件，会切分成若干个大小不超过SEGSIZE的小文件。该值的设置参考系统大文件的最大值。单位G。2的N次幂
--with-wal-blocksize 指定WAL日志文件的大小,单位M。2的N次幂。设置区间 1 - 1024M
--without-readline 避免使用Readline库（以及libedit）。这个选项禁用了psql中的命令行编辑和历史， 因此我们不建议这么做
--without-zlib	避免使用Zlib库。这样就禁用了pg_dump和 pg_restore中对压缩归档的支持。这个选项只适用于那些没有这个库的少见的系统
```

编译安装
```shell
make && make install
```

### 初始化数据库

执行下面的命令对数据库进行初始化，数据库的初始化需要切换到`postgres`用户
```shell
su postgres
/usr/local/pgsql/bin/initdb -D /home/postgres/data
```

创建systemd配置文件`/etc/systemd/system/pgserver.service`
```shell
[Unit]
Description=PostgreSQL database server
Documentation=man:postgres(1)

[Service]
Type=notify
User=postgres
ExecStart=/usr/local/pgsql/bin/postgres -D /home/postgres/data
ExecReload=/bin/kill -HUP $MAINPID
KillMode=mixed
KillSignal=SIGINT
TimeoutSec=0

[Install]
WantedBy=multi-user.target
```

配置开机启动并启动PostgreSQL
```
systemctl enable pgserver --now
```

编辑`~/.bashrc`添加下面数据
```shell
alias psql="/usr/local/pgsql/bin/psql"
alias pg_dump="/usr/local/pgsql/bin/pg_dump"
alias pg_restore="/usr/local/pgsql/bin/pg_resotre"
```

使其生效
```shell
source ~/.bashrc
```

使用默认`postgres`用户登录
```shell
psql -p 5432
```

修改数据库用户`postgres`密码
```shell
\password
```

### 连接配置

修改`/home/postgres/data/postgresql.conf`配置文件，设置监听地址
```shell
listen_addresses  = "*"
```

修改`/home/postgres/data/pg_hba.conf(host-based authentication file)`配置连接方式（添加）
```shell
host    all    all    0.0.0.0/0    trust
```

修改配置后重启数据库服务即可
```shell
systemctl restart pgserver
```
