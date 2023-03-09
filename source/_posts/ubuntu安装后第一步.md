---
title: ubuntu安装后第一步
top: false
cover: false
toc: true
mathjax: true
date: 2023-03-09 17:34:19
password:
summary: ubuntu安装后第一步
tags: os
categories: ubuntu
---




`无法解析域名cn.archive.ubuntu.com`

```
sudo service network-manager stop
```

```
sudo rm /var/lib/NetworkManager/NetworkManager.state
```

```
sudo service network-manager start
```

```
sudo apt-get install yum
```

```
sudo yum install vim
```

`Ubuntu下服务器开启远程连接`

```
sudo apt install openssh-server
```

```
service sshd status
```

`Ubuntu系统使用root远程登录`

```
sudo passwd root
```

```
vim /etc/ssh/sshd_config
```

	PermitRootLogin yes

***

