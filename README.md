# jackhai9's blog

这是一个基于 ``GitHub Pages`` 和 ``Hexo`` 搭建的静态博客，点击[这里](https://jackhai9.github.io)查看博客。

搭建过程或许有那么点小繁琐(其实只需要几个简单命令)，但一旦搭建完成，写文章是极简单，极舒服的。怎么个舒服法？

```bash
$ hexo n  #开写
$ hexo g  #生成
$ hexo d  #部署，可与hexo g合并为hexo d -g
```

## 搭建和使用(Windows下)：

### 准备工作
1. 安装Node.js，安装Git，还需要有GitHub账号。
2. 建立仓库 `<你的 GitHub 用户名>.github.io`，添加SSH公钥可以参考[这里](https://docs.github.com/zh/authentication/connecting-to-github-with-ssh)。
3. 以上完成后，使用`$ ssh -T git@github.com`验证是否成功。如出现Error: Permission denied (publickey)，则点[这里](https://docs.github.com/zh/authentication/troubleshooting-ssh/error-permission-denied-publickey)。

### 搭建和初始化
1. 在Node.js安装目录下安装Hexo：`$ npm install -g hexo`
2. 在喜欢的地方新建一个目录（建议目录名为`<你的 GitHub 用户名>.github.io`），用来初始化博客和存放文章。
3. 进入新建的目录（以后的命令基本上都在这个目录下）执行：`$ hexo init`

好啦，至此，搭建和初始化工作已经完成！

### 本地启动和预览
1. 生成静态页面：`$ hexo generate`
2. 本地启动：`$ hexo server`

浏览器输入 http://localhost:4000 就可以看到效果。

### 写文章和部署
1. 新建文章：`$ hexo new [layout] "postName"` 执行new命令后，生成指定名称的文章至hexo\source_posts\postName.md。
2. 用喜欢的编辑器(Typora)打开文件 hexo\source_posts\postName.md 开始尽情书写吧！关于markdown语法，可以参考[这里](https://markdown.com.cn/editor/)。
3. 写完后，`$ hexo server`  然后访问localhost:4000预览效果。(退出server用Ctrl+c)。
4. 然后 `$ hexo deploy` 同步到github。访问博客看看效果。 关于deploy，可以参考[这里](https://hexo.io/zh-cn/docs/commands#deploy)。
