title: 转投Hexo--使用Hexo在GitHub写博客
date: 2014-11-06 18:50:40
categories: Hexo 
tags: [Hexo] 
---
之前自搭[WordPress](http://jackhai.wordpress.com/)和Jekyll，感觉不够geek，虽然第一次用Jekyll的时候感觉cool毙了(累觉不爱)。更早之前使用[点点](http://jackhai.diandian.com/)、[新浪博客](http://blog.sina.com.cn/u/2483023881)、[cnblogs](http://www.cnblogs.com/jackhai9/)、[bloger](http://jackhai9.blogspot.com/)都只是浅尝辄止，现在还记得使用Windows Live Writer写新浪博客和博客园时那种惊艳的感觉。之后就把大部分东西都整理到个人笔记上了，用过有道、[为知](https://note.wiz.cn/i/57dec783)、Evernote，现在主力是为知，不怎么用印象，毕竟为知跟Chrome的组合已经让我习惯了，懒得换了。。
<!--more-->
# Hexo
我只能说，这是个B格极高的程序猿写作方式，正合我意。如果你也纠结于要去哪里写博客，那就来GitHub吧，你懂得。

hexo出自台湾大学生[tommy351](https://twitter.com/tommy351)之手，是一个基于Node.js的静态博客程序，其编译上百篇文字只需要几秒。hexo生成的静态网页可以直接放到GitHub Pages，BAE，SAE等平台上。先看看tommy是如何吐槽Octopress的 →＿→[Hexo 颯爽登場！](http://zespia.tw/blog/2012/10/11/hexo-debut/)

* 如果你对默认配置满意，只需几个命令便可秒搭一个hexo
* 如果你跟我一样喜欢折腾下，30分钟也足够个性化。
* 如果你过于喜欢折腾，可以折腾个把星期，尽情的玩。

搭建过程你或许觉得有那么点小繁琐(其实只需要几个简单命令)，但一旦搭建完成，写文章是极简单，极舒服的。怎么个舒服法？

```
$ hexo n  #开写 
```

```
$ hexo g  #生成 
```

```
$ hexo d  #部署，可与hexo g合并为hexo d -g 
```

### 安装和使用(Windows下)：

### 一、准备工作

1. 安装Node.js，安装Git，还需要GitHub账号。
2. 建立与你的GitHub用户名对应的仓库，仓库名必须为『your_user_name.github.com』，添加SSH公钥 [这是方法](https://help.github.com/articles/generating-ssh-keys/)。
3. 以上完成后，使用 
```
ssh -T git@github.com
```
验证是否成功。如出现Error: Permission denied (publickey)，则[点这里](https://help.github.com/articles/error-permission-denied-publickey/)。


### 二、安装及初始化

1. 在nodejs安装目录下使用cmd命令安装Hexo：
```
npm install -g hexo
```
2. 在你喜欢的地方新建一个目录，用来初始化你的博客和存放你以后的文章。
3. 进入你新建的目录(以后的命令基本上都在这个目录下执行)，打开cmd：
```
hexo init
```

好啦，至此，全部安装工作已经完成！


### 三、本地启动,预览下

1. 生成静态页面：
```
hexo generate
```
2. 本地启动：
```
hexo server
```

浏览器输入[http://localhost:4000](http://localhost:4000)就可以看到效果。


### 四、写文章

1. 新建文章
```
hexo new [layout] "postName" 
```

执行new命令后，生成指定名称的文章至hexo\source\_posts\postName.md。

2. 用你喜欢的编辑器打开文件 hexo\source\_posts\postName.md 开始尽情书写吧。。。关于markdown语法，可以参考[这里](http://wowubuntu.com/markdown/)。

3. 写完后，
```
hexo server
```
然后访问[localhost:4000](localhost:4000)预览效果。(退出server用Ctrl+c)。然后
```
hexo deploy
```
同步到github。访问网站看看效果。 关于deploy，可以参考[这里](http://hexo.io/docs/deployment.html)。

