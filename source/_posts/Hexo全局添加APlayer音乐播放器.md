---
title: Hexo全局添加APlayer音乐播放器
categories:
  - 环境搭建
tags:
  - 阿里云
  - Hexo
abbrlink: bd04265d
date: 2019-03-18 16:12:57
---

## 相关站点

+ [APlayer](https://aplayer.js.org/#/)
+ [APlayer Github](https://github.com/MoePlayer/APlayer)
+ [Hexo-Tag-Aplayer](https://github.com/MoePlayer/hexo-tag-aplayer)
+ [音乐直链搜索工具](https://music.liuzhijin.cn/)
<!--more-->
## 基于 Yilia 主题全局添加 APlayer 音乐播放器

编辑文件 hexo-theme-yilia/layout/_partial/left-col.ejs ，在文件的末尾追加以下代码；其中歌曲的歌词文件、封面图片、URL都可以从通过[音乐直链搜索工具](https://music.liuzhijin.cn/)获取，有些音乐由于版权得问题，个人建议还是通过下载到本地比较好。当然如果你的网站是部署再github上就不建议使用本地音乐，会存在卡顿的情况。如果是放在云服务器上，直接下载到本地。提示，如果下面的代码不能将APlayer播放器固定到理想的页面位置，可自行修改 div 标签的样式和 APlayer 的 fixed 参数值。

```js
<% if(theme.aplayer.enable) { %>
<link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/aplayer@1.10.1/dist/APlayer.min.css">
// 这里div的样式是笔者自己修改过的，你可以直接使用APlayer官方的原配置：<div id="aplayer"></div>
<div id="aplayer" style="position:absolute;left;0;bottom:0;"></div>
<script src="https://cdn.jsdelivr.net/npm/aplayer@1.10.1/dist/APlayer.min.js"></script>
<script src="https://cdn.jsdelivr.net/npm/color-thief-don@2.0.2/src/color-thief.js"></script>
<script>
  const ap = new APlayer({
    container: document.getElementById('aplayer'),
    autoplay: false, //自动播放
    listFolded: true, //播放列表默认折叠
    listMaxHeight: 90, //播放列表最大高度
    order: 'list', //音频循环顺序, 可选值: 'list', 'random'
    loop: 'all', //音频循环播放, 可选值: 'all', 'one', 'none'
    theme: '#e9e9e9', //切换音频时的主题色，优先级低于audio.theme
    preload: 'none', //音频预加载，可选值: 'none', 'metadata', 'auto'
    mutex: true, //互斥，阻止多个播放器同时播放，当前播放器播放时暂停其他播放器
    lrcType: 3, //歌词格式，可选值：3（LRC文件歌词格式），1（JS字符串歌词格式）
    volume: 0.7, //默认音量，请注意播放器会记忆用户设置，用户手动设置音量后默认音量即失效
    fixed: false, //吸底模式（fixed:true），迷你模式（mini:true），普通模式（注释此行或者设置fixed:false）
    audio: [{
        name: '平凡之路',
        artist: '朴树',
        lrc: '/downloads/lrc/平凡之路-朴树.lrc',
        cover: 'http://p2.music.126.net/W_5XiCv3rGS1-J7EXpHSCQ==/18885211718782327.jpg?param=300x300',
        url: 'http://fs.open.kugou.com/cd5cbe8edb012e4f77b0857cefc0956e/5c66accf/G097/M08/0A/1F/AYcBAFkQGpOAMUpuAEm-3SlWMyk951.mp3'
      },
      {
        name: '后会无期',
        artist: 'G.E.M.邓紫棋',
        lrc: '/downloads/lrc/后会无期-G.E.M.邓紫棋.lrc',
        cover: 'http://p1.music.126.net/vpvPajo3kn88nHc7jUjeWQ==/5974746185758035.jpg?param=300x300',
        url: 'http://m10.music.126.net/20190215193113/e5afc8b5376136029366f2053cf30f85/ymusic/2c87/6ec3/582e/0d572dcc04f8de34133c0f364b74c30c.mp3'
      }
    ]
  });

  //实现切换音频时，根据音频的封面图片自适应主题色
  const colorThief = new ColorThief();
  const setTheme = (index) => {
    if (!ap.list.audios[index].theme) {
      colorThief.getColorAsync(ap.list.audios[index].cover, function(color) {
        ap.theme(`rgb(${color[0]}, ${color[1]}, ${color[2]})`, index);
      });
    }
  };
  setTheme(ap.list.index);
  ap.on('listswitch', (data) => {
    setTheme(data.index);
  });
</script>
<% } %>
```

## 配置 Yilia 主题

编辑 Yilia 主题的配置文件 hexo-theme-yilia/_config.yml，在文件末尾追加以下内容。

```yml
aplayer:
  enable: true
```

如果你使用了yilia主题的相册功能，加了播放器后会出现原有相册显示不了的问题。当把hexo-tag-aplayer 配置好并且用几个页面测试后，发现相册功能失效了，查找问题后发现在ins.js中自动加了下面这些代码导致的失效。

```js
<link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/aplayer@1.10/dist/APlayer.min.css">
<script src="https://cdn.jsdelivr.net/npm/aplayer@1.10/dist/APlayer.min.js"></script>
<script src="https://cdn.jsdelivr.net/npm/meting@1.2/dist/Meting.min.js"></script>
```

经过一番在网上寻找后无果后，于是决定仔细研读官方文档，查看官方的中文文档后发现，可能是插件的自动脚本插入功能导致的，使得ins.js中自动插入了代码，于是关闭该功能试试：

```yml
aplayer:
  asset_inject: false
```

这段配置要加载博客根目录的配置文件中，而不是主题的配置文件。

## 重新编译 Hexo

提示，若音频文件使用的是本地资源文件，同时通过“hexo server”提供Web服务，那么则将无法通过APlayer的进度条调节播放进度，此时需要使用Nginx、Apache等Web服务器。

```shell
# 进入博客的根目录
# cd /blogroot

# 通过Hexo清理Public目录
# hexo clean

# 通过Hexo构建静态文件
# hexo generate

# 通过Hexo启动服务
# hexo server
```

## 最终效果图

{% asset_img 2020-03-18-16-17-08.png %}
