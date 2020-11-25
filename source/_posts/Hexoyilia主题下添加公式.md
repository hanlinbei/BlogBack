---
title: Hexo添加公式
categories:
  - 环境搭建
tags:
  - Hexo
  - 公式
toc: true
abbrlink: 53706
date: 2020-05-29 21:51:57
---

使用LaTex添加公式到Hexo博客里

<!--more-->

## 安装Kramed

hexo 默认的渲染引擎是 marked，但是 marked 不支持 mathjax。，所以需要更换Hexo的markdown渲染引擎为hexo-renderer-kramed引擎，后者支持mathjax公式输出

```shell
npm uninstall hexo-renderer-marked --save
npm install hexo-renderer-kramed --save
```

{% asset_img 2020-05-29-20-23-55.png %}

## 第二步：更改文件配置

打开/node_modules/hexo-renderer-kramed/lib/renderer.js，更改：

```js
// Change inline math rule
function formatText(text) {
    // Fit kramed's rule: $$ + \1 + $$
    return text.replace(/`\$(.*?)\$`/g, '$$$$$1$$$$');
}

为，直接返回text

// Change inline math rule
function formatText(text) {
    return text;
}
```

{% asset_img 2020-05-29-20-26-59.png %}

## 第三步: 停止使用 hexo-math，并安装mathjax包

卸载hexo-math

```shell
npm uninstall hexo-math --save
```

安装 hexo-renderer-mathjax 包

```shell
npm install hexo-renderer-mathjax --save
```

{% asset_img 2020-05-29-20-27-57.png %}
{% asset_img 2020-05-29-20-28-03.png %}

## 第四步: 更新 Mathjax 的 配置文件

打开/node_modules/hexo-renderer-mathjax/mathjax.html
注释掉script代码，并把以下代码复制到对应位置

```js
<script src="https://cdnjs.cloudflare.com/ajax/libs/mathjax/2.7.1/MathJax.js?config=TeX-MML-AM_CHTML"></script>
```

{% asset_img 2020-05-29-20-31-08.png %}

## 第五步: 更改默认转义规则

因为LaTeX与markdown语法有语义冲突，所以 hexo 默认的转义规则会将一些字符进行转义，所以我们需要对默认的规则进行修改.
打开/node_modules\kramed\lib\rules\inline.js

```js
escape: /^\\([\\`*{}\[\]()#$+\-.!_>])/,
```

更改为

```js
escape: /^\\([`*\[\]()# +\-.!_>])/,
```

```js
em: /^\b_((?:__|[\s\S])+?)_\b|^\*((?:\*\*|[\s\S])+?)\*(?!\*)/,
```

更改为

```js
em: /^\*((?:\*\*|[\s\S])+?)\*(?!\*)/,
```

{% asset_img 2020-05-29-20-32-27.png %}

## 第六步: 开启mathjax

打开/themes/yilia主题目录下的config.yml文件
因为我用的yilia主题，所以路径是/themes/yilia

我们需要在config.yml文件 中开启 Mathjax， 找到 mathjax 字段添加如下代码：(不同的主题配置方法略微有区别)

```shell
mathjax: true
```

{% asset_img 2020-05-29-20-33-09.png %}

需要注意的是：，无论是配置文件还是博客文件，配置项跟配置参数均有有一个空格，否则会配置失败

```shell
mathjax: true（mathjax:空格true）
而不是
mathjax:true（mathjax:true）
```

写博客文件时，要开启 Mathjax选项，， 添加以下内容：

```shell
mathjax: true
```

例如

```shell
title: Cplex求解器
categories:
  - 算法
tags:
  - Matlab
  - 算法
date: 2020-05-28 21:51:57
mathjax: true
```

如下图所示

{% asset_img 2020-05-29-20-35-37.png %}

通过以上步骤，我们就可以在 hexo 中使用 Mathjax 来书写数学公式

效果展示：
{% asset_img 2020-05-29-20-36-30.png %}


{% asset_img 2020-05-29-20-37-00.png %}
