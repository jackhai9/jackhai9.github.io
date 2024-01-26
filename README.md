# Hexo + GitHub Actions + GitHub Pages

仓库名：`<你的 GitHub 用户名>.github.io`，我这里是jackhai9.github.io
Markdown编辑器：推荐Typora

整体流程：克隆仓库、本地写Markdown文件、本地Hexo编译并预览效果、提交Hexo源文件到GitHub的source分支、触发GitHub Actions自动执行并部署到GitHub Pages

具体流程：

1. 克隆仓库：
   克隆当前仓库的source分支到本地：`git clone -b source --recurse-submodules git@github.com:jackhai9/jackhai9.github.io.git && cd jackhai9.github.io` 
   -- 因为项目里包含子模块（主题使用了别人的git项目），所以使用--recurse-submodules选项，这将会自动初始化并更新仓库中的每个子模块
   -- git clone -b 命令在大多数情况下应该自动设置跟踪分支。如果后续在 git pull 或者 git push 时提示 `There is no tracking information for the current branch.`，手动设置将当前分支与远程的source分支关联：git branch --set-upstream-to=origin/source source

3. 本地写Markdown文件：
   在本地的 jackhai9.github.io/source/_posts 目录下创建并书写MD文件：手动创建 或者 使用`$ hexo new [layout] "MD文件名称"`命令由Hexo自动创建

4. 本地Hexo编译并预览效果：
   在本地安装Hexo（为了能在本地预览，建议安装）：`npm install -g hexo-cli`   
   安装项目依赖：`npm install`  
   编译并启动一个本地的web服务器：`hexo server` 【会进行编译，不需要每次都手动运行`hexo generate`了】

5. 提交Hexo源文件到GitHub的source分支：
   一旦完成了MD文章的编辑或者其他配置的修改，就可以将这些Hexo源文件推送到source分支：

- git add .
- git commit -m "添加或更新文章/修改xx配置等"
- git push

5. 触发GitHub Actions自动执行并部署到GitHub Pages：
   当你推送更改到source分支后，GitHub Actions会自动执行定义好的工作流程（见jackhai9.github.io\.github\workflows\hexo-deploy.yml），这通常包括：安装NodeJS、安装项目依赖、安装全局的Hexo CLI、使用Hexo命令进行编译生成静态网站文件、并将这些文件推送到用于GitHub Pages的分支（比如master或gh-pages）




如何使用此仓库？

- 如果没有在[本地搭建静态博客生成器](#本地搭建静态博客生成器)，则需要先搭建；
- 如果已经在[本地搭建静态博客生成器](#本地搭建静态博客生成器)，则克隆本仓库到本地，

搭建过程或许有那么点小繁琐（其实就几个简单命令），一旦搭建完成，写文章极简单、极舒服。怎么个舒服法？

```bash
$ hexo n  #开写
$ hexo g  #生成
$ hexo d  #部署，可与hexo g合并为hexo d -g
```
点击[这里](https://jackhai9.github.io)查看最终生成的博客；

然后再 push Markdown文章到此仓库做备份。

当需要更换主题时：
1. 以子模块的方式下载主题：`git submodule add git@github.com:hexojs/hexo-theme-light.git themes/light`
2. 修改_config.yml中的theme: light
3. `ga .` 然后`gc ''`然后 `git push`

----------------------

# 本地搭建静态博客生成器

基于 [GitHub Pages](https://pages.github.com/) 和 [Hexo](https://hexo.io/zh-cn/) 搭建静态博客生成器，步骤如下：

## 准备工作

1. 安装Node.js，安装Git，还需要有GitHub账号。
2. 建立仓库 `<你的 GitHub 用户名>.github.io`，添加SSH公钥可以参考[这里](https://docs.github.com/zh/authentication/connecting-to-github-with-ssh)。
3. 以上完成后，使用`$ ssh -T git@github.com`验证是否成功。如出现Error: Permission denied (publickey)，则点[这里](https://docs.github.com/zh/authentication/troubleshooting-ssh/error-permission-denied-publickey)。


## 搭建和使用

### 搭建和初始化
1. 在Node.js安装目录下安装Hexo：`$ npm install -g hexo`
2. 在喜欢的地方新建一个目录（建议目录名为`<你的 GitHub 用户名>.github.io-hexo`），用来初始化博客和存放文章。
3. 进入新建的目录（以后的命令基本上都在这个目录下）执行：`$ hexo init`

好啦，至此，搭建和初始化工作已经完成！

### 本地启动和预览
1. 生成静态页面：`$ hexo generate`
2. 本地启动：`$ hexo server`  浏览器输入 http://localhost:4000 就可以看到效果。
3. 安装 [light](https://github.com/hexojs/hexo-theme-light) 主题

### 写文章和部署
1. 新建文章：`$ hexo new [layout] "postName"` 执行new命令后，生成指定名称的文章至 `<你的 GitHub 用户名>.github.io-hexo\source\_posts\postName.md`
2. 用喜欢的markdown编辑器（Typora）打开文件 `<你的 GitHub 用户名>.github.io-hexo\source\_posts\postName.md` 开始尽情书写吧！关于markdown语法，可以参考[这里](https://markdown.com.cn/editor/)。
3. 写完后，`$ hexo server`  然后访问localhost:4000预览效果。（退出server用Ctrl+c）
4. 然后 `$ hexo deploy` 同步到github。访问博客看看效果。 关于deploy，可以参考[这里](https://hexo.io/zh-cn/docs/commands#deploy)。
