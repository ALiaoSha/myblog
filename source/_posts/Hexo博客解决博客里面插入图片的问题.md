---
title: Hexo博客解决博客里面插入图片的问题
date: 2017-03-12 16:53:23
tags: Hexo
categories: IT技术
---
# 解决的问题
基于hexo和github搭建的博客有时候需要在博文中插入图片，用markdown语法`![](图片路径)`的方法只能在本地看到。不会被deploy到github pages上。这个问题目前有很多解决的方法。下面列出两种：
# 用CDN服务
[CDN](http://baike.baidu.com/item/CDN)(即内容分发网络)服务，可以让我们把图片放到服务器上。想在博客里面插入图片的时候，引用服务器上图片的链接地址就可以了。最简单的我们可以把图片上传到自己的个人微博，然后用`![](微博图片地址)`的方式引用图片。还有一个方法是使用[七牛](https://www.qiniu.com/)作为图床，每次写博客的时候先把图片上传到七牛的服务器上面去，需要的时候引用服务器上图片的地址。
# 本地图片上传到github page上
我们的博客是用hexo生成的和部署到github上面的，所以手动添加图片到github page上面特别麻烦。还好有一个插件`hexo-asset-image`可以帮助我们完成这个任务。  
安装插件
```
$ npm install hexo-asset-image --save
```
在全局_config.yml文件中plugins选项下加上`hexo-asset-image`表明要使用这个插件。  
之后你用hexo新建文章的时候会发现`_post`目录下回自动生成一个和文章名字一样的目录。文章所需要引用的图片可以都放在这个目录下面。在markdown文档里面引用的时候只要写`![title](文章名字/图片名字)`就可以顺利插入图片了。并且当用`hexo d`部署到github上面的时候，这些图片相应也会被上传上去。
# References
http://www.jianshu.com/p/c2ba9533088a
