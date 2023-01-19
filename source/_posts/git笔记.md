---
title: git笔记
top: false
cover: false
toc: true
mathjax: true
date: 2023-01-19 15:07:32
password:
summary: Git工具使用
tags: git
categories: 工具
---
## Git（文件只有放入了暂存区之后才会被追踪）
### 特点
-	版本控制:解决多人同时开发的代码问题，解决找回历史代码的问题
-	分布式:Git是分布式版本控制系统，如同一个GIT仓库，可以分不到不同的机器上，工作原理:找一台电脑做服务器（中央服务器），不关机，其他的人可以把这个服务器克隆到自己的电脑上

### 安装Git
```shell
sudo apt-get install git
```
#### 创建版本库
```shell
git init
```
#### 创建版本(两步)
```shell
git add 文件

git commit -m ‘版本说明’ # 版本:第一次创建版本的时候是记录里面的内容，以后的版本都是保存被修改的信息
```
#### 查询版本记录信息
```shell
 git log

 git log --pretty=oneline # 更好用

 git log --graph --pretty=oneline # 可以查看分支图
```
#### 版本的回退
>	有一个指针HEAD指向最新的版本， 上一个版本可以被表示为
	
> HEAD^,一个^表示前一个版本，100个^表示第前100个版本,还可以表示为:HEAD~1,HEAD~100,分别表示前一个和前100个版本
 
> 版本回退命令

```shell
 git reset –hard HEAD~1
```
> 版本前进命令

```shell
git reset –hard 版本编号 #（commit) 版本编号可以通过git reflog来查看，这里面都是操作记录
```

### 工作区
就是使用git init命令的目录，创建文件和编辑文件都是在工作区完成的
版本库:拥有暂存区,HEAD区
```shell
git add . # 就是添加修改到暂存区,git commit就是创建版本

git status # 可以查看工作区的情况

git add # 可以同时添加很多个文件（也可以是目录）到暂存区，用空格分别，并且使用git commit的时候可以将同时提交的所有文件，创建一个版本，

git checkout – 文件名 # 在没有加到暂存区可以直接用这个命令丢弃修改(删除以后也可用这个恢复)，如果已经加到了暂存区，可以通过git reset HEAD 文件名解除添加到暂存区，然后用这个命令直接放弃修改
```



#### 删除文件
rm 文件名,因为是对工作区的文件的修改，所以可以用git checkout恢复，也可以用git rm将删除放入暂存区，彻底删除。
```text
第一步：使用rm
第二步：如果要恢复,git checkout --文件名,如果要删除，git rm放入暂存区
第三步: 如果删除文件的修改被放入了暂存区，可以用git reset HEAD 文件名 解除暂存，然后用git checkout恢复
```



#### git 分支的概念(比如两个平行的宇宙，出现了交点)
```shell
git主分支:master分支,HEAD指向master然后master再指向提交的指针
git从分支:dev分支，和master分支相互独立，可以和master分支合并，还可以删除

git branch可以查看有多少个分支
创建分支: git branch 名字
创建新的分支并切换到新分支: git checkout -b dev（名字随便取）,创建了dev分支，HEAD就会指向dev
切换到master分支: git checkout master
将dev分支的东西合并到master（一般都是合并在master分支上，也可以合并到其他的相关分支上，比如修复hug的时候）：git merge dev
删除dev分支：git branch -d dev
```
>  分支合并冲突:master分支和其他分支都修改了同一个文件的版本，合并就会报错，解决办法就是手动解决 

>  分支的管理决策: 在两个分支中修改了不同的文件的版本，那么合并的时候，就不会是快速合并,而是跳出一个弹框,第一行写入你的说明信息，退出之后合并完成，或者git merge --no-ff -m 合并版本说明 合并的分支， 这样可以禁用快速合并，保留分支的文件信息

>  分支bug管理:当我们在一个分支上进行工作的时候，其他的分支有bug需要及时的修复，那么可以使用git stash保存正在工作的分支上的状态，然后马上去需要修复bug的分支上修复bug：
-	首先我们要确定在哪一个分支上修复bug 
-	然后在到那个分支上创建一个临时的分支来修复这个bug 
-	然后切换回出hug的分支
-	然后合并分支，并用 git merge --no-ff -m '说明信息' 临时分支,用来保存bug记录
-	然后删除临时分支，在修复完成以后,再回到最初工作的分支
-	用git stash pop恢复工作现场就可以继续工作（用git stash list 可以查看保存的工作现场的列表)




#### 链接远程的仓库
```text
生成ssh公钥:ssh-keygen -t rsa -C'749062870@qq.com'一直回车(3个)

生成的ssh存放在用户文件夹下面的.ssh下面

将.ssh下面的id_rsa.pub内容复制到github

使用ssh -T git@github.com来测试是否连通
```


#### GitHub：
```text
1、创建新的仓库(同git中的git init): start project  => 项目名称 => 公开 => 两个add(readme使用说明，gitignore忽略文件)
2、添加ssh账户(本机和github交互):创好仓库之后，点击头像下拉，找到settings => SSH and GPG keys => New SSH key就可以添加本地ssh账户，Title随便写,Key则需要本地的ssh公钥，（生成本地电脑的ssh公钥，回到家目录，编辑vim .gitconfig,然后ssh-keygen -t rsa -C '749062870@qq.com'一直回车，完成之后，cd ssh，然后cat id_rsa.pub，复制过去就行了）
3、克隆项目:github中的仓库中，code下拉中，选择ssh(我们用的就是ssh)，复制地址到相对应的文件夹下用终端 git clone 复制的地址, 将github中的项目下载到本地
4、推送代码:被下载到本地的项目,不用master分支开发，而是创建一个自己的分支，一直在自己的分支上开发，开发完成之后先add，commit到本地，然后可以用git push origin 分支名称，就可以推送到github上面
5、追踪远程分支：git branch --set-upstream-to=origin/远程分支 本地分支,追踪之后，就可以用git status来查看本地分支和远程分支的提交情况
6、拉去代码: git pull origin 要拉取的分支.从远程分支上下载，拉取代码

```

> 链接——>切换分支-->add--->commit---->push到lqs

##### 常用命令

```shell
git config --global user.name '用户名' # 修改配置的用户名
git config --global user.email '邮箱'  # 修改配置的邮箱

git init # 初始化git仓库
git status # 查看工作区的文件是否被管理
git add . # 把工作区的文件提交到暂存区，也就是将工作区的文件进行管理
git commit -m '日志' # 将暂存区的文件提交到本地仓库
git log # 查看提交的日志，点击q退出
git branch # 查看本地分支
git branch -a # 查看本地和远程的分支
git checkout '远程分支' # 可以从本地分支切换到远程分支
git checkout -b '分支名称' # 新建并切换到分支
git branch -d '分支名称' # 删除分支，分支被合并之后才允许删除
git branch -D '分支名称' # 删除分支，强制删除
git merge '分支名称' # 合并分支
git push '远程地址 '# 将本地仓库推送到远程仓库或者如下
git remote add origin '远程地址' # 给远程仓库添加别名
git push -u origin '分支名称' # 远程仓库没有这个分支，第一次向远程仓库推送分支
git push  # 第二次推送
git clone '远程地址' # 克隆远程仓库的代码
git pull '远程地址' '远程分支' # 拉去远程仓库的代码
```