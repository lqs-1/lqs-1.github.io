---
title: Linux安装nginx
top: false
cover: false
toc: true
mathjax: true
date: 2023-05-10 06:44:53
password: 5c4aaf271c7c13b2372e84982f82f2cd279026950438dfc769c95dcabdfd2a87
summary: Linux Install Nginx
tags: nginx
categories: Linux
---
## Linux Install Nginx

### 安装之前

> 安装编译工具

```shell
yum install -y gcc-c++
```

> 创建nginx安装目录

```shell
mkdir /usr/local/nginx # nginx安装到这里
mkdir /usr/local/nginx/package # 需要的依赖包放在这里 如果要全编译安装的话就需要这个文件夹
mkdir /home/nginxInstallPackage # 创建nginx安装包或者依赖包目录
```
### 下载相关依赖
> 全编译安装下载依赖
```shell
cd /home/nginxInstallPackage
# 下载openssl 支持nginx开通https的模块 非对称加密的工具(https://www.openssl.org/) 选择性下载
wget http://nobibibi.top/somg/file/nginx/openssl-1.1.1l.tar.gz && tar -zxvf openssl-1.1.1l.tar.gz -C /usr/local/nginx/package/
# 下载zlib 是一款支持zlib压缩的模块(zlib.net) 必须下载
wget http://nobibibi.top/somg/file/nginx/zlib-1.2.11.tar.gz && tar -zxvf zlib-1.2.11.tar.gz  -C /usr/local/nginx/package/
# 下载pcre 是一个支持正则表达式的模块(www.pcre.org) 必须下载
wget http://nobibibi.top/somg/file/nginx/pcre-8.42.tar.gz && tar -zxvf pcre-8.42.tar.gz  -C /usr/local/nginx/package/
```
> 命令安装依赖
```shell
yum -y install pcre pcre-devel zlib openssl openssl-devel
```

### 安装Nginx

#### 全编译安装

> 下载nginx

```shell
cd /home/nginxInstallPackage

wget http://nobibibi.top/somg/file/nginx/nginx-1.20.2.tar.gz && tar -zxvf nginx-1.20.2.tar.gz -C /usr/local/nginx/package/ # 下载nginx
```

> 执行`configure`

```shell
cd /usr/local/nginx/package/nginx-1.20.2

./configure --prefix=/usr/local/nginx \
	--with-http_ssl_module --with-openssl=../openssl-1.1.1l \
    --with-pcre=../pcre-8.42 \
    --with-zlib=../zlib-1.2.11 # 执行
```

> 编译并安装nginx

```shell
make && make install
```

#### 命令安装依赖方式编译安装
> 下载nginx

```shell
cd /home/nginxInstallPackage

wget http://nobibibi.top/somg/file/nginx/nginx-1.20.2.tar.gz && tar -zxvf nginx-1.20.2.tar.gz -C /usr/local/nginx/package/ # 下载nginx
```

> 执行`configure`

```shell
cd /usr/local/nginx/package/nginx-1.20.2

./configure --prefix=/usr/local/nginx # 执行
```

> 编译并安装nginx

```shell
make && make install
```

### nginx结构
安装后所有的文件都在`/usr/local/nginx`下面

> `sbin`存放nginx命令

```shell
cd sbin

./nginx # 启动 

./nginx -s reload # 重启

./nginx -s stop # 关闭 

pkill -9 nginx # 强制关闭
```

> `conf`

```shell
cat nginx.conf # nginx默认配置
```

> `html`是静态文件目录


### 其他安装方式

#### 通过命令行安装
```shell
apt install nginx 
/etc/init.d/nginx start # 启动 
/etc/init.d/nginx stop # 停止 
/etc/init.d/nginx restart # 重启
```

#### 通过docker安装

> 获取镜像

```shelll
docker pull nginx
```

> 运行镜像

```shell
docker run -itd -p 80:80 --name nginx --restart always nginx
```

> 进入容器执行命令

```shell
docker exec -it nginx /bin/bash -c "一个或者多个命令" # 不进入容器执行命令

docker exec -it nginx /bin/bash # 进入容器执行命令
```

> 复制

```shell
docker cp /home/default.conf nginx:/etc/nginx/conf.d/ # 将宿主机的文件复制到容器内
docker cp nginx:/etc/nginx/conf.d/default.conf /home/ # 将容器中的文件复制到宿主机
```

> 容器管理

```
docker start nginx   # 启动
docker stop nginx	 # 停止
docker restart nginx # 重启
```

> 修改容器配置

```shell
docker container update --restart=always nginx
docker restart nginx # 重启
```
