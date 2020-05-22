---
title: VsCode编写Hexo支持的MD文档
categories:
  - 环境搭建
tags:
  - Vscode
  - Hexo
  - 随笔
abbrlink: c057518d
date: 2019-03-12 12:12:57
---

## vscode编写markdown

markdown是用hexo写博客的可选文本格式之一，通常我们用hexo new xxx来创建一篇新的post。md编辑器有很多，预览我们hexo项目中的md文章的方法也有很多，我的方案是编辑与预览都在vscode中完成。

这里要先提一下在hexo3.0版本中编写md文件时新的图片插入方式：只要在_config.yml中添加post_asset_folder: true，就会在生成新post时创建一个同名文件夹；在md中输入{% asset_img xxx %}，就可以插入这个文件夹中的图片。(这种方式较之以前把图片放在source/images，方式更整洁，图片随文章分类)
vscode有很多md的插件，这里推荐两个插件，并针对hexo做一些配置调整，以满足插入图片与预览md的需求：
<!--more-->
1. 粘贴图片Paste Image
{% asset_img 2020-03-13-13-13-00.png %}
这个插件用来在md文档中粘贴图片，默认会在文档的同级目录下新建一个图片文件，并在md中插入一行相对路径的图片代码。迎合上述hexo的新图片插入方式，可以在vscode的user-settings里新增两条配置：

```shell
"pasteImage.path": "${currentFileNameWithoutExt}/",
"pasteImage.insertPattern": "{% asset_img ${imageFileName} %}"
```

这样以来，粘贴的图片就会保存到md文档的同名文件夹下，文档中将插入hexo asset语法的代码。
2. 预览Markdown Preview Enhanced
{% asset_img 2020-03-13-13-14-28.png %}
这个是下载量最高的vscode md预览插件，支持很多功能，并支持扩展md解析语法。现在就要利用这个功能来解决一个问题：vscode内无法预览代码的图片。ctrl+shift+P输入Markdown Preview Enhanced: Extend Parser调出插件的parse.js文件，修改其中的onWillParseMarkdown方法：

```js
module.exports = {
  onWillParseMarkdown: function(markdown) {
    return new Promise((resolve, reject)=> {
      markdown = markdown.replace(
        /\{%\s*asset_img\s*(.*)\s*%\}/g,
        (whole, content) => (`![](${content})`)
      )
      return resolve(markdown)
    })
  },
  ...
}
```

这样以来，我们md中的代码就会在解析预览时被替换成md的图片语法，并且同样采用相对路径，图片预览成功。
