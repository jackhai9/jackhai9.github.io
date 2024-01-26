title: 使用gitcafe同步github
date: 2014-11-21 19:50:53
categories: [折腾]
tags:
---
**注意：2016年6月开始 gitcafe 已迁移至 Coding.net**

从监控宝和百度统计来看，我github上搭建的blog访问还是慢，在gitcafe上搞个备份。

有点复杂，记录下步骤，防止遗忘：

1.在[gitcafe](https://gitcafe.com/)注册账号、创建同名项目等步骤，同github一样，[见此](http://jackhai9.github.io/2014/11/06/转投hexo-使用hexo在github写博客/)。

~~2.**在github和gitcafe上同时上传博客的时候，不能使用相同的秘钥。**~~

<!--more-->

cd到home目录( C:\Users\JackHai)下的.ssh目录，
`ssh-keygen -t rsa -C "jackhai789@gmail.com" `  回车/回车/回车
以上命令执行后会在当前目录下生成id_rsa的秘钥和id_rsa.pub的公钥，把公钥内容copy到你的gitcafe账号的“SSH公钥管理”里，然后在命令行里使用命令 ssh -T git@gitcafe.com测试下，出现 “Hi jackhai9! You've successfully authenticated, but GitCafe does not provide shell access.”  提示，则ok了！

~~这个时候回到HexoBlog目录，执行hexo g，然后upblog.bat，哈哈，同时部署到github和gitcafe，ok！good！~~

注意：
* ~~因为我的git不是单独安装的，我是使用的github软件里的git，在.ssh目录里有github_rsa和github_rsa.pub,我之前就是直接把github_rsa.pub这个公钥放到gitcafe里，所以会出现access denied的情况，连接不上。重新生成秘钥放到gitcafe里就ok了。~~
* ~~因为gitcafe和github上传时要求所在的目录不一致(github是HexoBlog这个根目录，而gitcafe是.deploy目录)，所以理论上不能一键上传到两个上(需要cd切换目录)，因此我写了upblog.bat这个批处理文件，有了它，只要在HexoBlog根目录下运行`upblog.bat`这个命令就可以了。upblog.bat里面的内容是：`cd .deploy/ && git checkout gitcafe-pages && git push -u origin gitcafe-pages && git checkout master && cd .. && hexo d`~~


## 终极解决方法：
**思路是：先配置_config.yml文件的deploy，使得一个`hexo d`命令能把我们的项目部署到两个仓库里；再自动每次deploy完一个后，删除.deploy文件夹，再自动deploy另一个。**

两次需要的.deploy文件夹是不一样的，可能是里面的配置有不一样的地方，所以我的解决方法是：在部署完一个后就删除掉.deploy文件夹，再重新部署，这就要求我们的_config.yml文件里对两个仓库都要进行配置，配置如下：

    deploy:
      type: git
      repository: 
        gitcafe: git@gitcafe.com:jackhai9/jackhai9.git,gitcafe-pages
        github: git@github.com:jackhai9/jackhai9.github.com.git,master

* 注意":"后面的空格，还有，不要使用tab！！总之，就是在英文状态下编辑这个文件，不要使用tab，不然deploy的时候会报错！

实现一个`hexo d`命令自动部署到两个平台，我的方法是：使用windows下的批处理bat文件，我依然使用upblog.bat这样一个名字(当然你可以随意命名)，文件内容超级简单：`hexo d && rd .deploy && hexo d`

Ok搞定！（rd 命令就是删除文件夹）

既然如此，索性把`hexo g`命令也一起放入批处理中吧，实现真正的一键搞定。upblog.bat文件内容更加超级简单：`hexo d -g && rd .deploy && hexo d`
这样，编辑完md文件后，直接在根目录`upblop.bat`即可！
