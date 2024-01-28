# Hexo + GitHub Actions + GitHub Pages

## 项目说明

- 有两个分支：source分支：编译前的源码文件。master分支：编译后的静态文件。
- 仓库名的格式要求：`<你的GitHub用户名>.github.io`，我这里是jackhai9.github.io，就会自动开启 Github Pages 功能。
- 当然，你自己搭建的时候如果想使用其他分支或其他仓库名，也可以。相信你可以自己解决:smirk:

## 整体流程

1. 手动 -- 克隆仓库
2. 手动 -- 本地写Markdown文件
3. 手动 -- 本地Hexo编译并预览效果
4. 手动 -- 提交Hexo源文件到GitHub的source分支
5. 自动 -- 触发GitHub Actions自动执行[workflow](https://github.com/jackhai9/jackhai9.github.io/blob/source/.github/workflows/hexo-deploy.yml)最终部署到GitHub Pages

## 概念说明

- [Hexo](https://github.com/hexojs/hexo) ：静态站点生成器，依赖 [Node.js](https://nodejs.cn/)。快速、简洁且高效的博客框架，很适合用来搭建博客。
- [GitHub Actions](https://docs.github.com/en/actions)：GitHub 提供的一种自动化工具，会去执行你创建的 workflow。
- [workflow](https://docs.github.com/en/actions/using-workflows/about-workflows)：工作流，即 按希望的顺序排列组合一系列指令（action）。这些指令可以是各种任务，如安装依赖、测试代码、部署应用等。GitHub Actions 使用 .github/workflows 下面的 YAML 文件去定义 workflow。
- [GitHub Pages](https://pages.github.com/)：GitHub 提供的一项服务，它允许你从 GitHub 仓库直接托管静态网站。常用于个人、项目或组织的网站，并且是免费的，不用再单独去申请域名了，当然也支持指向你已申请的域名。很适合用来托管博客、项目文档、个人简历等内容。GitHub Pages 也默认使用 [Jekyll](https://github.com/jekyll/jekyll) 这样的静态站点生成器，直接从 Markdown 文件生成网站内容，见[我的另一个博客](https://github.com/jackhai9/blog)。总之，GitHub Pages为用户提供了一种简单便捷的方式，用于将代码和文档以网页的形式分享给他人。

## 具体流程：

1. 克隆仓库：

   - 克隆当前仓库的source分支到本地：`git clone -b source --recurse-submodules git@github.com:jackhai9/jackhai9.github.io.git && cd jackhai9.github.io`

   > 因为项目里包含子模块（其他的git项目），所以使用 `--recurse-submodules`选项，将初始化并更新仓库中的每个子模块
   >
   > `git clone -b <分支名称>` 命令在大多数情况下应该会自动设置跟踪分支。如果后续在 `git pull`或 `git push`时提示 `There is no tracking information for the current branch.`，则需要手动将本地的 source 分支与远程的 source 分支关联：git branch --set-upstream-to=origin/source source

2. 本地写Markdown文件：

   - 在本地的 jackhai9.github.io/source/_posts 目录下创建并书写MD文件：手动创建 或者 使用 `$ hexo new [layout] "MD文件名称"`命令由Hexo自动创建

3. 本地Hexo编译并预览效果：

   - 在本地安装Hexo（为了能在本地预览，建议安装）：`npm install -g hexo-cli`
   - 安装项目依赖：`npm install`
   - 编译并启动一个本地的web服务器：`hexo server` 【会进行编译，不需要每次都手动运行 `hexo generate`了】

4. 提交Hexo源文件到GitHub的source分支：

   - 一旦完成了MD文章的编辑或者其他配置的修改，就可以将这些Hexo源文件推送到source分支：

   ```bash
   git add .
   git commit -m "添加或更新文章/修改xx配置等"
   git push
   ```

5. 触发GitHub Actions自动执行[workflow](https://github.com/jackhai9/jackhai9.github.io/blob/source/.github/workflows/hexo-deploy.yml)最终部署到GitHub Pages：

   - 当你推送更改到source分支后，GitHub Actions会自动执行定义好的工作流程（见jackhai9.github.io\.github\workflows\hexo-deploy.yml），这通常包括：

   ```bash
   切到source分支，安装Hexo依赖的环境(NodeJS、项目依赖、)并安装Hexo，运行Hexo的编译指令会把当前source分支的源码编译成静态文件输出到public目录，再用[actions-gh-pages](https://github.com/peaceiris/actions-gh-pages)把public目录中的静态文件提交到master分支并部署到 GitHub Pages 页面
   ```



对于GitHub Pages, 你有两种常见的分支选项来发布你的网站：

1. **`master`或`main`分支**：对于用户或组织的GitHub Pages网站（如`<你的GitHub用户名>.github.io`），通常直接使用`master`或`main`分支来发布。
2. **`gh-pages`分支**：对于项目仓库，你可以使用特别的`gh-pages`分支来托管GitHub Pages内容。

GitHub提供了选项来选择哪个分支用于GitHub Pages，这可以在仓库的设置中配置。这样你可以根据你的项目需求选择最适合的分支。

触发GitHub提供的workflow“pages-build-deployment”，

## 其他

- Markdown编辑器：推荐Typora



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
3. `ga .` 然后 `gc ''`然后 `git push`

---

# 本地搭建静态博客生成器

基于 [GitHub Pages](https://pages.github.com/) 和 [Hexo](https://hexo.io/zh-cn/) 搭建静态博客生成器，步骤如下：

## 准备工作

1. 安装Node.js，安装Git，还需要有GitHub账号。
2. 建立仓库 `<你的 GitHub 用户名>.github.io`，添加SSH公钥可以参考[这里](https://docs.github.com/zh/authentication/connecting-to-github-with-ssh)。
3. 以上完成后，使用 `$ ssh -T git@github.com`验证是否成功。如出现Error: Permission denied (publickey)，则点[这里](https://docs.github.com/zh/authentication/troubleshooting-ssh/error-permission-denied-publickey)。

## 搭建和使用

### 搭建和初始化

1. 在Node.js安装目录下安装Hexo：`$ npm install -g hexo`
2. 在喜欢的地方新建一个目录（建议目录名为 `<你的 GitHub 用户名>.github.io-hexo`），用来初始化博客和存放文章。
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
