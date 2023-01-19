---
title: ngonx+fdfs搭建
top: false
cover: false
toc: true
mathjax: true
date: 2023-01-19 16:23:26
password:
summary: nginx+fdfs搭建
tags: 
	-nginx
    -fdfs
categories: 工具
---
## Nginx+FastDfs
[hfdfs项目](https://github.com/happyfish100)     fdfs和那llibfastcommon依赖还有fdfs-ngx-module还有nginx链接模块都可以直接克隆相对应仓库

### 安装nginx及fastdfs-nginx-module

#### 解压缩 fastdfs-nginx-module-master.zip

```shell
sudo ./configure --prefix=/usr/local/nginx/ --add-module=fastdfs-nginx-module-master解压后的目录的绝对路径/src
```

#### 如果nginx还没安装执行
```shell
sudo ./make
sudo ./make install
```
```shell
./configure --with-http_ssl_module --with-openssl=../openssl-1.1.1m  --with-pcre=../pcre-8.42 --with-zlib=../zlib-1.2.11 --prefix=/usr/local/nginx/ --add-module=/usr/share/fastdfs/fastdfs-nginx-module-master/src
```

#### 如果nginx安装了执行
> sudo ./make

```shell
sudo cp fastdfs-nginx-module-master解压后的目录中src下的mod_fastdfs.conf  /etc/fdfs/mod_fastdfs.conf
```
> sudo vim /etc/fdfs/mod_fastdfs.conf

```text
修改内容：

connect_timeout=10

tracker_server=自己ubuntu虚拟机的ip地址:22122

url_have_group_name=true

store_path0=/home/python/fastdfs/storage
```

> sudo cp 解压缩的fastdfs-master目录中的http.conf  /etc/fdfs/http.conf

> sudo cp 解压缩的fastdfs-master目录中的mime.types /etc/fdfs/mime.types

> sudo vim /usr/local/nginx/conf/nginx.conf

> 在http部分中添加配置信息如下(反向代理)：

```shell
server {

        listen       8888;

        server_name  localhost;

        location ~/group[0-9]/ {

            ngx_fastdfs_module;

        }

        error_page   500 502 503 504  /50x.html;

        location = /50x.html {

        root   html;

        }

}
```
#### 启动nginx
```shell
sudo /usr/local/nginx/sbin/nginx
```

> 在换ip之后，storage.conf和client.conf还有mod_fastdfs.cong


#### python使用fdfs

```shell
pip install fdfs_client-py-master.zip

>>> from fdfs_client.client import Fdfs_client

>>> client = Fdfs_client('/etc/fdfs/client.conf')

>>> ret = client.upload_by_filename('test')

>>> ret
```
```
{'Groupname':'group1','Status':'Uploadsuccessed.','Remotefile_id':'group1/M00/00/00/wKjzh0_xaR63RExnAAAaDqbNk5E1398.py','Uploadedsize':'6.0KB','Localfilename':'test', 'Storage IP':'192.168.243.133'}
    ```