---
title: V免签部署
top: false
cover: false
toc: true
mathjax: true
date: 2023-12-28 15:35:03
password: 5c4aaf271c7c13b2372e84982f82f2cd279026950438dfc769c95dcabdfd2a87
summary: 免签支付
tags: pay
categories: pay
---

## V免签

---

### 资源地址

[`v免签官方源码`](https://github.com/szvone/vmqphp)

[`v免签安装监控端`](https://pub-pce.oss-cn-chengdu.aliyuncs.com/public/tools/base.apk
)

### 部署步骤

> - 安装宝塔面板并安装 `nginx1.18`、`php7.3`、`mysql5.6`

> - 创建php项目绑定域名创建目录	将源码解压到项目目录下 再将运行目录设置为public

> - 伪静态选择`thinkphp`

> - 默认文档中将`index.html`设置为第一个

> - 打开网站路径`/config/database.php` 设置好你的mysql账号密码并保存

> - 在数据库中导入源码包中的`vmq.sql`

> - 登录网站的用户名密码默认都是admin

> - 手机上安装监控端就可以了 这样可以根据V免签的`api`对接项目