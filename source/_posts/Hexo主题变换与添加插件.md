---
title: Hexo主题变化与添加插件
categories:
  - 环境搭建
tags:
  - 阿里云
  - Hexo
  - 随笔
abbrlink: c7a10181
date: 2019-03-12 12:12:57
---
# 更换主题Theme及基本配置

## 更换主题

默认主题是landscape，在themes文件夹下，可以使用别人开发好的主题，这里有很多，我使用的是这一个<https://github.com/litten/hexo-theme-yilia>
下载之后放到themes文件夹下即可：git clone git@github.com:litten/hexo-theme-yilia.git
<!--more-->
## 主题基本配置

配置在_config.yml文件中，基本的配置尝试一下就知道了，不在给出
图片的位置
比如打赏的支付宝二维码图片，是在当前主题下的source/assets/img/下 （不是当前博客根目录）
{% asset_img 2020-03-12-20-51-10.png %}
配置：

```shell
# 打赏基础设定：0-关闭打赏； 1-文章对应的md文件里有reward:true属性，才有打赏； 2-所有文章均有打赏
reward_type: 1
# 打赏wording
reward_wording: '谢谢你请我吃糖果'
# 支付宝二维码图片地址，跟你设置头像的方式一样。比如：/assets/img/alipay.jpg
alipay: /assets/img/alipay.jpg
# 微信二维码图片地址
weixin: /assets/img/weixin.png
```

## 文章评论设置

很多默认的评论插件要么不维护了，要么需要翻墙的。不过有基于github开发的gittalk还是不错的。Gitalk 是一个基于 Github Issue 和 Preact 开发的评论插件。使用 Github 帐号登录，界面干净整洁，最喜欢的一点是支持 MarkDown语法

主要特性：

使用 Github 登录
支持多语言 [en, zh-CN, zh-TW, es-ES, fr]
支持个人或组织
无干扰模式（设置 distractionFreeMode 为 true 开启）
快捷键提交评论 （cmd|ctrl + enter）
支持MarkDown语法

在layout/_partial/post目录下新增gitalk.ejs文件

```js
<div id="gitalk-container" style="padding: 0px 30px 0px 30px;"></div>

<link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/gitalk@1/dist/gitalk.css">
<script src="https://cdn.jsdelivr.net/npm/gitalk@1/dist/gitalk.min.js"></script>
<script type="text/javascript">

if(<%=theme.gitalk.enable%>){
 var gitalk = new Gitalk({
   clientID: '<%=theme.gitalk.ClientID%>',
   clientSecret: '<%=theme.gitalk.ClientSecret%>',
   repo: '<%=theme.gitalk.repo%>',
   owner: '<%=theme.gitalk.githubID%>',
   admin: ['<%=theme.gitalk.adminUser%>'],
   id: '<%= page.date %>',
   distractionFreeMode: '<%=theme.gitalk.distractionFreeMode%>'
})
gitalk.render('gitalk-container')
}
</script>
```

修改source-src/css/目录下comment.scss文件

```css
#disqus_thread, .duoshuo, .cloud-tie-wrapper, #SOHUCS, #gitment-ctn, #gitalk-container {
 padding: 0 30px !important;
 min-height: 20px;
}

#SOHUCS {
 #SOHU_MAIN .module-cmt-list .block-cont-gw {
  border-bottom: 1px dashed #c8c8c8 !important;
 }
}
```

在layout/_partial目录下的article.ejs文件内新增gitalk相关的配置代码

```js
<% if (theme.gitalk.enable){ %>
       <div id="gitalk-container"></div>
       <%- include post/gitalk.ejs %>
  <% } %>
```

最后在yilia主题配置文件中新增gitalk相关的配置：

```yml
#gitalk评论
gitalk:
  enable:  true
  githubID: 写自己github的ID
  repo: 新建存放评论的仓库名
  ClientID:  下面讲述如何书写 需要注册 OAuth Application
  ClientSecret:  下面讲述如何书 需要注册 OAuth Application
#不这样书写容易报错
  adminUser: "['仓库名','仓库名']"
  labels: gitalk
  perPage: 15
  pagerDirection: last
  createIssueManually: true
  distractionFreeMode: true
```

当别人评论你的文章时，会需要它是授权。点击<https://github.com/settings/applications/new>

进行注册。注册界面如下。
{% asset_img 2020-03-12-21-04-54.png %}
红色方框写的是博客地址就 ok 了！！
注册成功后，会获取到 Client ID/scerct 。如下图所示：
{% asset_img 2020-03-12-21-05-53.png %}
最终演示
{% asset_img 2020-03-12-21-06-26.png %}

## 网站访问量显示

我使用了不蒜子第三方的统计插件，网址：<http://ibruce.info/2015/04/04/busuanzi/>
在themes\yilia\layout\_partial下的footer.ejs中加入如下代码即可

```js
<script async src="//dn-lbstatics.qbox.me/busuanzi/2.3/busuanzi.pure.mini.js">
</script>
<span id="busuanzi_container_site_pv">
  本站总访问量<span id="busuanzi_value_site_pv"></span>次
</span>
<span id="busuanzi_container_site_uv">
总访客数<span id="busuanzi_value_site_uv"></span>人次
</span>
```

## 添加字数统计和阅读时长功能

1.安装 hexo-wordcount
在博客目录下打开Git Bash Here 输入命令

```shell
npm i --save hexo-wordcount
```

2.文件配置
在theme\yilia\layout\_partial\post下创建word.ejs文件：

```js
<div style="margin-top:10px;">
    <span class="post-time">
      <span class="post-meta-item-icon">
        <i class="fa fa-keyboard-o"></i>
        <span class="post-meta-item-text">  字数统计: </span>
        <span class="post-count"><%= wordcount(post.content) %>字</span>
      </span>
    </span>

    <span class="post-time">
      &nbsp; | &nbsp;
      <span class="post-meta-item-icon">
        <i class="fa fa-hourglass-half"></i>
        <span class="post-meta-item-text">  阅读时长: </span>
        <span class="post-count"><%= min2read(post.content) %>分</span>
      </span>
    </span>
</div>
```

然后在 themes/yilia/layout/_partial/article.ejs中添加
```js

<div class="article-inner">
    <% if (post.link || post.title){ %>
      <header class="article-header">
        <%- partial('post/title', {class_name: 'article-title'}) %>
        <% if (!post.noDate){ %>
        <%- partial('post/date', {class_name: 'archive-article-date', date_format: null}) %>
        <!-- 需要添加的位置 -->
        <!-- 开始添加字数统计-->
        <% if(theme.word_count && !post.no_word_count){%>
          <%- partial('post/word') %>
          <% } %>
        <!-- 添加完成 -->

        <% } %>
      </header>
```

3.开启功能
在站点的_config.yml中添加下面代码

```yml
# 是否开启字数统计
#不需要使用，直接设置值为false，或注释掉
word_count: True
```

## 添加背景音乐

1. 打开网易云音乐首页，然后搜索你要添加的背景音乐<http://music.163.com/>
{% asset_img 2020-03-12-21-16-59.png %}
2. 搜索到歌曲点击生成外链播放器，进去下一个界面
{% asset_img 2020-03-12-21-17-30.png %}
3. 复制外链播放器的代码
打开yilia主题下的`_partial`文件夹下的`left-col.ejs`文件复制文件内容到最下端笔者添加了一些判断和表达式

```js

<!-- 网易云音乐插件 -->
<% if (theme.music && theme.music.enable){ %>
    <div style="position:absolute; bottom:120px left:auto; width:85%">
        <iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width="240" height="52" src="//music.163.com/outchain/player?type=2&id=<%=theme.music.id||1334445174%>&auto=<%=theme.music.autoplay?1:0%>&height=32"></iframe>
    </div>
<% } %>
```

{% asset_img 2020-03-12-21-20-12.png %}
4. 在yilia主题配置文件中添加属性

```yml
# 网易云音乐插件
music:
  enable: true
  #id: 1332647902  # 网易云分享的ID
  autoplay: true  # 是否开启自动播放
```

## 写作的一些说明

执行命令：hexo new "xxxx"创建Markdown文件，在博客的source\_posts文件夹下
比如如下例子，
comments设置为true允许评论，若设置为false则不能评论
reward设置为true允许打赏，若设置为false则不能打赏，（注意对应主题的配置文件reward_type: 设置的为1）
{% asset_img 2020-03-12-21-10-01.png %}
