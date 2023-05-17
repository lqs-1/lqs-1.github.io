---
title: GitHubPages部署Hexo
top: true
cover: false
toc: true
mathjax: true
date: 2023-05-17 21:33:38
password: 5c4aaf271c7c13b2372e84982f82f2cd279026950438dfc769c95dcabdfd2a87
summary: GitHubPages部署Hexo
tags: hexo
categories: 博客部署
---

## GitHubPages部署Hexo

### 创建仓库
<你的github用户名>.github.io

### 安装Git
[`点击安装git工具`](https://git-scm.com/)

> 配置Git
```shell
git config --global user.name "你的GitHub用户名"
git config --global user.email "你的GitHub注册邮箱"
```

> 生成ssh密钥文件
```shell
ssh-keygen -t rsa -C "你的GitHub注册邮箱"
```
然后直接三个回车即可，默认不需要设置密码
然后找到生成的.ssh的文件夹中的id_rsa.pub密钥，将内容全部复制
打开GitHub_Settings_keys 页面，新建new SSH Key

### 安装Node.js
[`点击安装node.js`](https://nodejs.org/en/download)
配置环境变量自行百度

### 安装Hexo
> 安装Hexo
```shell
npm install -g hexo-cli 
```

> 初始化博客(只是安装之后使用，多端的时候不用)
```shell
hexo init blog
```

### 安装Hexo博客推送依赖
```shell
sudo npm install hexo-deployer-git --save
```

### 配置Hexo推送
> 编辑博客主目录的`_config.yml`
```shell
deploy:
  type: git
  repository: 仓库克隆地址
  branch: main
```

> 生成博客静态文件
```shell
hexo g
```

> 推送博客到github
```shell
hexo d
```

> 本地测试
```shell
hexo s
```

