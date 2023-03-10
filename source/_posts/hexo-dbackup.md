---
title: hexo-dbackup
top: true
cover: false
toc: true
mathjax: true
date: 2023-01-16 05:51:25
password: 5c4aaf271c7c13b2372e84982f82f2cd279026950438dfc769c95dcabdfd2a87
summary: hexo数据备份，换机方案
tags: hexo
categories: 博客部署
---

<div align="middle">
<iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=330 height=86 src="//music.163.com/outchain/player?type=2&id=1997527356&auto=1&height=66"></iframe>
</div>

## 换机操作前

> 在仓库中创建一个hexo分支并设置为默认

> 在本地配置好ssh连接，并测试是否连通github

> git clone 这个仓库，因为设置成默认分支，所以直接clone的就是hexo

> 将hexo分支下除了.git的文件全部删除，再将源文件中除了.deploy_git的所有文件拷贝到这里，如果安装了主题，需要将主题中的.git文件夹删除。原理就是hexo的源码和配置不会上传到github，只是上传的.deploy_git里面通过hexo g这个命令生成的静态文件，.deploy_git里面的文件会上传到哪里，这个你在项目目录_config.yaml中配置的，这里配置了上传到哪里，上传到哪个分支，因为这个.deploy_git是可以根据hexo g来生成，所以不用上传,即是说： `新拉下来的这个仓库是存放源代码的，如果是第一次做这个操作，需要将其他的文件全部删除，只留下一个.git文件夹，这是仓库信息，然后将源代码里面的除了.deploy_git的所有数据全部复制过来`

> 添加一个.gitignore文件(可选)
```yaml
.DS_Store
Thumbs.db
db.json
*.log
node_modules/
public/
.deploy*/
```

> git add .

> git commit -m "desc"

> git push origin

---

## 换了电脑之后
> 配置好git的ssh连接

> 安装好Node

> sudo npm install hexo-cli-g

> git cloen 备份仓库

> cd xxx.github.io

> npm install

> npm install hexo-deployer-git --save

> hexo g

> hexo d

就可以开始写博客了

> 如果在已经编辑过的电脑上写博客，那么还需要git pull更新一下

## 多端操作
> 在新机子上clone仓库，clone的是源代码分支,然后安装依赖，开始写博客，如果新机子上以前有这个仓库，那么需要git pull更新一下再写

> 在clone的这个仓库中，可以直接写博客，不用删除什么，也可以直接hexo -g -d来将静态页面推送到github，但是每次写完了，静态代码推送完了，然后删除.deploy_git文件夹，最后通过git将源码上传到仓库。


## git配置ssh

> ssh-keygen -t rsa -C "邮箱" 通过这个来生成rsa密钥

> 生成的ras存放在用户目录下的.ssh文件夹中

> cat id_rsa.pub 将公钥填写到github

> ssh -T git@github.com 测试是否连上github