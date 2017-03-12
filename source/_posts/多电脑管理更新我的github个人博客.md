---
title: 多电脑管理更新我的github个人博客
date: 2017-03-12 10:08:29
tags: Hexo
category: IT技术
---

昨天跟着[Mungo](http://mungo.space/2015/10/14/create-hexo-on-github-4/)的博文实现多电脑终端管理我的github pages博客的时候，边做边记录下来了我遇到的一些问题和解决方案。结果在atom上写的时候忘记save了。直接关掉了atom发现刚刚写的markdown文档大小为0kb。我苍了个天！
以后写文档一定定时save！ MD!

# 1. 前言
用Hexo+github搭建了自己的个人博客以后，有时候需要在多台电脑上更新自己的博客。这里的解决方案是：把github当做hexo博客源代码的云端备份。想在哪台电脑上更新博客的时候直接在github上面拉取源代码下来编辑以后就可以把更新的博客deploy到github服务器。然后把更新的源代码也push到你放hexo博客源代码的repo上去。

    Notes：A电脑表示我最开始搭建hexo博客的电脑，  B电脑是我想在上面对博客进行更新的另外一台电脑。


# 在A电脑中从本地上传Hexo博客源码到Github仓库
首先在你的github上面新建一个repository，用来存放你的hexo博客源码。
我新建的命名myblog。
注意：新建repo的时候不要加README.md文件，不然待会push A电脑上的本地代码上去的时候会有 `refusing to merge unrelated histories`的报错。

## 1.初始化仓库

在Hexo博客的目录下运行Git Bash并输入以下命令：
```
$git init
$git remote add origin <server address>
```
这里<server address>指的是在线仓库的地址，比如在这里我的就应该是`https://github.com/ALiaoSha/myblog.git`，如果你用其它git仓库服务，填写对应仓库地址即可。
origin是本地分支,remote add会将本地仓库映射到Github仓库。

## 2.把本地文件同步到Github上面
分别输入执行以下命令：
```
$ git add .                      #添加所有目录，注意add后面有个点`.`
$ git commit -m "add to Github"  #添加提交说明，每次提交都需要
$ git push -u origin master      #把更新推送到云端
```
这时可以登录Github账户查看刚创建的myblog仓库中是否上传成功。


注意:
为了在另一台电脑上配置更加方便，严重建议把Hexo博客目录下_config.yml文件复制粘贴一份，并重命名为hexo_config.yml；把themes目录下你用到主题目录下的_config.yml文件也复制一份，并粘贴到博客根目录，注意，是’博客根目录’，并命名为theme_config.yml。原因是我们push代码的时候，我们自己安装的themes是从别人的github上clone下来的。也就是说你的一个git项目中又包含了一个git项目。那么被包含的那个项目是untracked的，被包含的主题目录并不能上传，所以我们需要把这两个配置文件都保存下来在进行同步工作。

# 在B电脑中从Github仓库取回Hexo到本地
如果你的第二台电脑是没有安装运行hexo所需要的环境的话，第一步先要把环境搭起来：安装好Git和Node.js
## 1.把myblog仓库里的文件取回B电脑本地

安装环境完成后，在新文件夹下运行Git Bash并分别执行以下几条命令：
```
$ git init
$ git remote add origin <myblog address>  # 关联github上的myblog仓库
$ git fetch –all                          # 取回源代码
$ git merge origin/master                 # 与本地分支合并
```
这里<myblog address>是你刚才新建的Giuhub上myblog仓库的地址。fetch是将仓库中的内容取出来。reset则是不做任何合并（merge）处理，直接把取出的内容保存。
运行完merge命令后你会发现文件夹中才会出现刚刚上传的内容。

## 3.配置新的Hexo

如果是新PC，不要忘记我们本机并没有安装Hexo博客
执行下面的指令在B电脑上安装hexo
```
$ npm install hexo --save
$ hexo init
$ npm intall
```
    Notes: 在B电脑上安装hexo的时候就不需要像在A电脑上面那样要先执行 `$ npm install -g hexo-cli`了

3.记得在第一篇中讲过，新安装的Hexo是没有hexo-deployer-git依赖包的，需要手动安装
```
$ npm install hexo-deployer-git --save   # 自动部署依赖包
$ npm install hexo-math --save           # mathjax渲染依赖包
```

5.安装主题，我在上文中提到新安装的主题并不能被上传，所以也需要重新手动安装(以NexT主题为例)

```
$ git clone https://github.com/iissnan/hexo-theme-next themes/next
```
这里要注意的是： themes/next 是主题保存目录。
我是用`$ git submodules add <address>`命令安装的主题，本来以后这样就可以把主题代码也push到myblog下面。后来发现push上去的git submodule是重定向到主题作者的repo的。
6.我们之前备份的两个配置文件hexo_config.yml和theme_config.yml有用了,hexo_config.yml重命名为_config.yml覆盖根目录下的同名文件，而theme_config.yml也重命名为_config.yml覆盖主题目录下的config.yml文件。注意文件名前面的下划线’_’。

7.输入命令`$ hexo g`和命令`$ hexo s`，然后在浏览器输入`localhost:4000`中进行预览。如果没有问题那么我们在B电脑上就配置成功了。

# 在B电脑上更新博客
现在在B电脑上也可以像在A电脑上一样更新博客了，同样是
```$ hexo new post "my_new_post"```
编辑完文章，然后执行`$ hexo generate`和`$ hexo deploy`就可以成功发表了。
这里`$ hexo deploy`命令是将我们的博客文章发表到我们的Github上的Hexo博客，并不是前文新建的blog仓库，新建的blog仓库用来保存我们的Hexo程序。

把B电脑上的Hexo从本地同步到Github仓库
当发表完文章，我们还需要把Hexo程序同步到我们Github的blog仓库。执行下面指令：

```
$ git add .
$ git commit -m "commit from PC_B"
$ git push -u origin master
```
成功后，我们再次把hexo博客源代码同步更新到了我们的Github仓库myblog。
如果再想用A电脑更新我们的博客，只需要在执行添加文章之前先把程序从myblog仓库拉取下来便可。输入命令：
```
$ git pull https://github.com/ALiaoSha/myblog.git
```
即可完成。

# 注意:
我们每次更新博客时，为了保持我们每次用到的程序都是最新的。

每次更新博客之前都需要执行
```
$ git pull https://github.com/xxxx/xxx.git保持本地最新；
```
每次更新博客之后都需要执行
```
$ git add .
$ git commit -m "message"
$ git push -u origin master
```
以保持Github仓库程序最新。

这样，我们就实现了在不同电脑都能对我们的Hexo博客进行维护了。
