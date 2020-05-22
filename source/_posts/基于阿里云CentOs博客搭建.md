---
title: 基于阿里云CentOs博客搭建
categories:
  - 环境搭建
tags:
  - 阿里云
  - Hexo
  - Ngnix
  - 随笔
abbrlink: 6f636c5a
date: 2019-03-11 12:12:57
---

# 基于CentOS 环境
总共就是客户端和服务端以及阿里云的配置
# 阿里云:
<!--more-->
{% asset_img 2020-03-11-10-04-12.png %}
{% asset_img 2020-03-11-10-08-59.png %}
配置完毕,端口一定要开放
# 第一部分: 服务器环境搭建，包括安装 Git 、Nginx配置 、创建 git 用户
安装git和nodejs
```shell
yum install git

#安装NodeJS
curl --silent --location https://rpm.nodesource.com/setup_5.x | bash -

```
```shell
adduser git
chmod 740 /etc/sudoers
vim /etc/sudoers

找到以下内容

## Allow root to run any commands anywhere
root    ALL=(ALL)    ALL

在下面添加一行

git ALL=(ALL) ALL
```
如图:
{% asset_img 2020-03-11-10-21-37.png %}
保存退出后改回权限
```shell
chmod 400 /etc/sudoers
```
随后设置Git用户的密码，
# 需要root权限
```shell
sudo passwd git
```
切换至git用户，创建 ~/.ssh 文件夹和 ~/.ssh/authorized_keys 文件，并赋予相应的权限
```shell
su git
mkdir ~/.ssh
vim ~/.ssh/authorized_keys
#然后将电脑中执行 cat ~/.ssh/id_rsa.pub | pbcopy ,将公钥复制粘贴到authorized_keys
chmod 600 ~/.ssh/authorzied_keys
chmod 700 ~/.ssh
```
然后再客户端(电脑上),执行ssh -v git@ip地址(就是你阿里云的外网地址) ,应该就能免密登陆了
# 接下来安装Nginx和配置
```shell
安装:
yum install nginx

启动:
1.systemctl start nginx
2.systemctl enable nginx
```
然后把服务器的公网地址输入浏览器, 出现下面的图片就对了,没出来就有问题
{% asset_img 2020-03-11-10-27-22.png %}
## 配置Nginx
```shell
vim /etc/nginx/conf.d/default.conf
```
依照下图进行修改，将“/usr/share/nginx/html”改为“/usr/share/nginx/html/blog”。
{% asset_img 2020-03-11-10-28-28.png %}
```shell
sudo mkdir -p /usr/share/nginx/html/blog
sudo chown -R git:git /usr/share/nginx/html/blog
```
这样上面的地址就算对应上了

在服务器上初始化一个git裸库
切换到git用户，然后切换到git用户目录，接着初始化裸库，代码如下：
```shell
su git
cd ~
git init --bare blog.git
```
接着新建一个post-receive文件
```shell
vim ~/blog.git/hooks/post-receive
```
然后在该文件中输入以下内容：
```shell
#！/bin/sh
git --work-tree=/usr/share/nginx/html/blog --git-dir=/home/git/blog.git checkout -f
```
保存退出之后，再输入以下代码，赋予该文件可执行权限。
```shell
chmod +x ~/blog.git/hooks/post-receive
```
# 第二部分: 本地Hexo初始化， 包括安装 NodeJS 、hexo-cli, 生成本地静态网站
##初始化Hexo博客
首先要安装 hexo-cli，安装hexo-cli 需要 root 权限，使用 sudo 运行
```shell
sudo npm install -g hexo-cli
```
然后初始化Hexo程序
```shell
cd ~/Documents
hexo init blog
```
等执行成功以后安装两个插件， hexo-deployer-git 和 hexo-server ,这俩插件的作用分别是使用Git自动部署，和本地简单的服务器。
[hexo-deployer-git帮助文档](https://github.com/hexojs/hexo-deployer-git)
[hexo-server帮助文档](https://hexo.io/zh-cn/docs/server.html)
```shell
cd blog
npm install hexo-deployer-git --save
npm install hexo-server
```
## 初始化Nodejs
```shell
brew install nodejs
```
生成自己的第一篇文章 hello world !
使用 hexo new <文章名称> 来新建文章，该命令会成成一个 .md文件放置在 sources/_posts文件夹。
```shell
hexo new "hello Hexo"
vim sources/_posts/hello-hexo.md
```
编辑完毕以后， 使用hexo g将 .md文件渲染成静态文件，然后启动hexo-server：
```shell
hexo g
hexo server
```
现在便可以打开浏览器访问 http://localhost:4000 来查看我们的博客了！
然后停掉

## 配置_config.yml,完成自动化部署
然后打开~/Documents/blog/_config.yml 找到 deploy
```xml
deploy:
    type: git
    repo: git@SERVER:/home/git/blog.git       #此处的SERVER需改为你自己服务器的ip
    branch: master                            #这里填写分支
    message:                                  #提交的信息
```
{% asset_img 2020-03-11-10-39-23.png %}
保存后，尝试将我们刚才写的"hello hexo"部署到服务器
```shell
hexo clean
hexo generate --deploy
```
访问服务器地址，就可以看到我们写的文章"Hello hexo",以后写文章只需要：
```shell
hexo new "Blog article name"
···写文章
hexo clean && hexo generate --deploy
```
