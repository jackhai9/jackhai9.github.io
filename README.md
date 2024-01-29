# Markdown + Hexo + GitHub Actions + GitHub Pages

## 项目说明

- 仓库名的格式要求：`<你的GitHub用户名>.github.io`，我这里是jackhai9.github.io，就会自动开启这个仓库的 Github Pages 功能。
- 有两个分支：source分支（存放编译前的源码文件）、master分支（存放编译后的静态文件）。
- 当然，你如果想用其他仓库名或者分支，也可以。相信你可以自己解决:smirk:
- **[这里](https://jackhai9.github.io)就是通过此方式创建的博客。**

## 整体流程

1. 手动 -- 克隆仓库
2. 手动 -- 本地安装Hexo并编译/预览效果
2. 手动 -- 本地写Markdown文件并编译/预览效果
4. 手动 -- 提交Hexo源文件到GitHub的source分支
5. 自动 -- 触发GitHub Actions执行[自定义的workflow](https://github.com/jackhai9/jackhai9.github.io/blob/source/.github/workflows/hexo-deploy.yml)最终部署到GitHub Pages

## 概念说明

- [Hexo](https://github.com/hexojs/hexo) ：静态站点生成器，依赖 [Node.js](https://nodejs.cn/)。快速、简洁且高效的博客框架，很适合用来搭建博客。
- [GitHub Actions](https://docs.github.com/en/actions)：GitHub 提供的一种自动化工具，会去执行你创建的 workflow。
- [workflow](https://docs.github.com/en/actions/using-workflows/about-workflows)：工作流，即 按希望的顺序排列组合一系列指令（action）。这些指令可以是各种任务，如安装依赖、测试代码、部署应用等。
- [GitHub Pages](https://pages.github.com/)：GitHub 提供的一项服务，它允许你从 GitHub 仓库直接托管静态网站。常用于个人、项目或组织的网站，并且是免费的，不用再单独去申请域名了，当然也支持指向你已申请的域名。很适合用来托管博客、项目文档、个人简历等内容。GitHub Pages 也默认使用 [Jekyll](https://github.com/jekyll/jekyll) 这样的静态站点生成器，直接把 Markdown 文件转成 HTML，见[我的另一个博客](https://github.com/jackhai9/blog)。总之，GitHub Pages为用户提供了一种简单便捷的方式，用于将代码和文档以网页的形式分享给他人。

## 具体流程

1. 克隆仓库

   克隆当前仓库的source分支到本地：`git clone -b source --recurse-submodules git@github.com:jackhai9/jackhai9.github.io.git && cd jackhai9.github.io`

   > 因为项目里包含子模块（其他的git项目），所以使用 `--recurse-submodules`选项，将初始化并更新仓库中的每个子模块

   > `git clone -b <分支名称>` 命令在大多数情况下应该会自动设置跟踪分支。如果后续在 `git pull`或 `git push`时提示 `There is no tracking information for the current branch.`，则需要手动将本地的 source 分支与远程的 source 分支关联：git branch --set-upstream-to=origin/source source

2. 本地安装Hexo并编译/预览效果

   1. 在本地安装Hexo（为了能在本地预览，建议安装）：`npm install -g hexo-cli`
   2. 安装项目依赖：`npm install`
   3. 启动一个本地的web服务器并预览效果：`hexo server` 【会进行编译，不需要每次都手动运行 `hexo generate`进行编译了】

4. 本地写Markdown文件并编译/预览效果

   1. 在本地的 jackhai9.github.io/source/_posts 目录下创建并书写MD文件：手动创建 或者 使用 `$ hexo new [layout] "MD文件名称"`命令由Hexo自动创建
   2. 启动web服务器并预览效果：`hexo server`，然后访问localhost:4000预览效果
   
4. 提交Hexo源文件到GitHub的source分支

   一旦完成了MD文章的编辑或者其他配置的修改，并本地预览没问题后，就可以将这些修改推送到source分支：

   > 注意配置 .gitignore，把 node_modules/、public/ 等排除

   ```bash
   git add .
   git commit -m "添加或更新文章/修改xx配置等"
   git push
   ```

5. 触发GitHub Actions执行[自定义的workflow](https://github.com/jackhai9/jackhai9.github.io/blob/source/.github/workflows/hexo-deploy.yml)最终部署到GitHub Pages

   当推送更改到source分支后，GitHub Actions会自动执行自定义的workflow（要求以YAML文件的形式放在 .github/workflows 下面），简单解释下自定义的workflow：

   1. 切到source分支
   2. 安装依赖(NodeJS、项目依赖)、安装Hexo
   3. 运行Hexo的编译指令把当前source分支的源码编译成静态文件输出到public目录
   4. 运行[actions-gh-pages](https://github.com/peaceiris/actions-gh-pages)把public目录中的静态文件提交到master分支并部署到 GitHub Pages
   
   > actions-gh-pages会创建workflow“pages-build-deployment”，会被GitHub Actions执行，最终把静态文件部署到GitHub Pages
   
   > 两个workflow都有完整的[执行日志](https://github.com/jackhai9/jackhai9.github.io/actions)
   >
   > - 第1个是我们自定义的workflow的[执行日志](https://github.com/jackhai9/jackhai9.github.io/actions/workflows/hexo-deploy.yml)
   >
   > - 第2个是actions-gh-pages创建的workflow的[执行日志](https://github.com/jackhai9/jackhai9.github.io/actions/workflows/pages/pages-build-deployment)

## 其他

- **一旦上述的流程在本地执行过一次之后，后续写博客只需要执行3、4就可以了。**

- Markdown编辑器：推荐Typora。

- 对于GitHub Pages，你有两种常见的分支选项来发布你的网站：

  - **`master`或`main`分支**：对于用户或组织的 GitHub Pages 网站（如`<你的GitHub用户名>.github.io`），通常直接使用`master`或`main`分支来发布。
  - **`gh-pages`分支**：对于项目仓库，你可以使用特别的`gh-pages`分支来托管 GitHub Pages 内容。

  GitHub提供了选项来选择哪个分支用于GitHub Pages，这可以在仓库的设置中配置。这样你可以根据项目需求选择最适合的分支。

- 为什么在本地安装Hexo仍然是个好主意？ 尽管GitHub Actions可以从“执行Hexo的安装”一直到“最终部署到GitHub Pages”，你不一定需要在本地安装Hexo，但本地安装Hexo仍然有几个好处：
  
  1. 本地预览：本地安装Hexo允许你在提交和推送之前预览你的博客，确保一切看起来都像你期望的那样。
  2. 更快的迭代：在本地进行更改和预览可以加快写作和编辑的过程，因为你不需要等待CI/CD流程完成来看到更改。
  3. 故障排除：如果出现构建错误或其他问题，本地安装可以帮助你更快地诊断和解决这些问题，否则需要等到发布时或发布后才知道有问题。
  
- 当需要更换博客主题时：
  
  1. 在GitHub上找到相应的Hexo主题后，建议fork到自己仓库，便于进行定制化修改，比如我所用的[主题](https://github.com/jackhai9/hexo-theme-light)；
  2. 以子模块的方式添加到themes目录下：`git submodule add git@github.com:jackhai9/hexo-theme-light.git themes/light`
  3. 修改_config.yml中的theme: light
  4. 提交修改：`git add .` 然后 `git commit -m "新增主题xxx"`然后 `git push`
  5. 如果你对子模块进行了定制化修改，则需要
     1. cd到子模块目录(比如themes/light目录)，提交修改到子模块的远程仓库；
     2. 再cd回主项目目录(比如jackhai9.github.io目录)，再提交修改到主项目的远程仓库；
  
- 通过SSH与GitHub进行免密交互，无需每次访问都使用用户名和密码，设置方式：
  
  [检查本地现有的SSH密钥](https://docs.github.com/zh/authentication/connecting-to-github-with-ssh/checking-for-existing-ssh-keys)、[生成本地新的SSH密钥](https://docs.github.com/zh/authentication/connecting-to-github-with-ssh/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent)、[添加本地SSH密钥到GitHub账户](https://docs.github.com/zh/authentication/connecting-to-github-with-ssh/adding-a-new-ssh-key-to-your-github-account)。



