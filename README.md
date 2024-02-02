# Markdown + Hexo + GitHub Actions + GitHub Pages

## 项目说明

- **[这里](https://jackhai9.github.io)就是通过此方式创建的博客。**
- 仓库名的格式要求：`<你的GitHub用户名>.github.io`，我这里是jackhai9.github.io，就会自动开启这个仓库的 Github Pages 功能。
- 有两个分支：source分支（存放编译前的源码文件）、master分支（存放编译后的静态文件）。
- 当然，你如果[想用其他仓库名或者分支](#更换仓库或分支)也可以。 😉

## 整体流程

1. 手动 -- [克隆仓库](#克隆仓库)
2. 手动 -- [本地安装Hexo](#本地安装Hexo)
2. 手动 -- [本地写Markdown文件并编译和预览](#本地写Markdown文件并编译和预览)
4. 手动 -- [提交Hexo源文件到GitHub的source分支](#提交Hexo源文件到GitHub的source分支)
5. 自动 -- [触发GitHub Actions执行自定义的workflow最终部署到GitHub Pages](#触发github-actions执行自定义的workflow最终部署到github-pages)

## 概念说明

- [Markdown](https://daringfireball.net/projects/markdown/)：一种易写易读的标记语言，通过解析器转为 XHTML (or HTML)。语法因不同的解析器或编辑器而异。
- [Hexo](https://github.com/hexojs/hexo) ：静态站点生成器，依赖 [Node.js](https://nodejs.cn/)。快速、简洁且高效的博客框架，很适合用来搭建博客。
- [GitHub Actions](https://docs.github.com/en/actions)：GitHub 提供的一种自动化工具，用于执行 workflow。
- [workflow](https://docs.github.com/en/actions/using-workflows/about-workflows)：工作流，即 按希望的顺序排列组合一系列指令（action）。这些指令可以是各种任务，如安装依赖、测试代码、部署应用等。
- [GitHub Pages](https://pages.github.com/)：GitHub 提供的一项服务，从 GitHub 仓库直接托管静态网站。常用于个人、项目或组织的网站，并且是免费的，不用再单独去申请域名，当然也支持指向你已申请的域名。很适合用来托管博客、项目文档、个人简历等。GitHub Pages 也默认使用 [Jekyll](https://github.com/jekyll/jekyll) 这样的静态站点生成器，直接把 Markdown 转成 HTML，见[我的另一个博客](https://github.com/jackhai9/blog)。在部署时会用到 GitHub Actions 提供的自动化构建和部署流程。总之，GitHub Pages为用户提供了一种简单便捷的方式，用于将代码和文档以网页的形式分享给他人。

## 具体流程

1. #### 克隆仓库

   克隆当前仓库的source分支到本地：`git clone -b source --recurse-submodules git@github.com:jackhai9/jackhai9.github.io.git && cd jackhai9.github.io`

   > 这个项目会依赖其他的git项目（依赖其他主题，我[没有使用Hexo默认的主题](#更换博客主题)），而且我是[以子模块的方式来引入的其他主题](#以子模块的方式添加到themes目录下)，所以需要使用 `--recurse-submodules`选项，此选项会初始化并更新仓库中的每个子模块。

   > 如果没有以子模块的方式来引入，比如是以“作为依赖库直接复制代码”的方式引入的，则不需要`--recurse-submodules`选项。更多依赖管理的方式看[这里](#依赖管理)

   > `git clone -b <分支名称>` 命令在大多数情况下应该会自动设置跟踪分支。如果后续在 git pull 或 git push 时提示 `There is no tracking information for the current branch.`，则需要手动将本地的 source 分支与远程的 source 分支关联：`git branch --set-upstream-to=origin/source source`

2. #### 本地安装Hexo

   1. 在本地安装Hexo（基于这个[原因](#为什么在本地安装Hexo仍然是个好主意)，还是建议在本地安装Hexo）：`npm install -g hexo-cli`

   2. 安装项目依赖：`npm install`

   3. 启动一个本地的web服务器：`hexo server` ，并能在浏览器访问 localhost:4000，说明安装成功

      > `hexo server`会进行编译，不需要每次都手动运行 `hexo generate`进行编译

4. #### 本地写Markdown文件并编译和预览

   1. 在当前项目的 source/_posts 目录下创建Markdown文件：手动去创建 或者 使用 `$ hexo new [layout] "MD文件名称"`命令由Hexo自动创建，然后愉快的编写Markdown文件吧
   2. 再次启动web服务器：`hexo server`，在浏览器访问localhost:4000预览博客效果吧
   
4. #### 提交Hexo源文件到GitHub的source分支

   一旦完成了Markdown文章的编写或其他配置的修改，并本地预览没问题，就可以将这些修改推送到source分支：

   > 注意配置 .gitignore，忽略 node_modules/、public/ 等

   ```bash
   git add .
   git commit -m "添加或更新文章/修改xx配置等"
   git push
   ```

5. #### 触发GitHub Actions执行自定义的workflow最终部署到GitHub Pages

   当推送更改到source分支后，因为已经在 .github/workflows/ 下面以YAML文件的形式自定义了[workflow]((https://github.com/jackhai9/jackhai9.github.io/blob/source/.github/workflows/hexo-deploy.yml))，GitHub Actions会自动执行这个workflow，简单解释下自定义的workflow在干啥：

   1. 切到source分支
   
   2. 安装依赖(NodeJS、项目依赖)、安装Hexo
   
   3. 运行Hexo的编译指令把当前source分支的源码编译成静态文件输出到public目录
   
   4. 运行[actions-gh-pages](https://github.com/peaceiris/actions-gh-pages)把public目录中的静态文件提交到master分支并部署到 GitHub Pages
   
      > actions-gh-pages会创建workflow“pages-build-deployment”，会被GitHub Actions执行，最终把静态文件部署到GitHub Pages
   
      >两个workflow都有完整的[执行日志](https://github.com/jackhai9/jackhai9.github.io/actions)
      >
      >- 第1个是我们自定义的workflow的[执行日志](https://github.com/jackhai9/jackhai9.github.io/actions/workflows/hexo-deploy.yml)
      >- 第2个是actions-gh-pages创建的workflow的[执行日志](https://github.com/jackhai9/jackhai9.github.io/actions/workflows/pages/pages-build-deployment)
   

## 其他说明

- **一旦上述的流程在本地执行过一次之后，后续写博客只需要执行3、4就可以了。**

- Markdown编辑器：推荐Typora。

#### 配置SSH

通过SSH与GitHub进行免密交互，无需每次访问都使用用户名和密码，设置方式：

[检查本地现有的SSH密钥](https://docs.github.com/zh/authentication/connecting-to-github-with-ssh/checking-for-existing-ssh-keys)、[生成本地新的SSH密钥](https://docs.github.com/zh/authentication/connecting-to-github-with-ssh/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent)、[添加本地SSH密钥到GitHub账户](https://docs.github.com/zh/authentication/connecting-to-github-with-ssh/adding-a-new-ssh-key-to-your-github-account)。

#### 为什么在本地安装Hexo仍然是个好主意

尽管GitHub Actions可以从“执行Hexo的安装”一直到“最终部署到GitHub Pages”，不是必须在本地安装Hexo，但本地安装Hexo仍然有几个好处：

1. 本地预览：本地安装Hexo允许你在提交和推送之前预览你的博客，确保一切看起来都像你期望的那样。
2. 更快的迭代：在本地进行更改和预览可以加快写作和编辑的过程，因为你不需要等待CI/CD流程完成来看到更改。
3. 故障排除：如果出现构建错误或其他问题，本地安装可以帮助你更快地诊断和解决这些问题，否则需要等到发布时或发布后才知道有问题。

#### 依赖管理

Git项目中引入其他Git项目的方式：

1. **子模块（Submodules）:** 将一个Git仓库作为另一个Git仓库的子模块。这是管理复杂项目的一种流行方式。
   - **适用场景：** 子模块适合于需要维持高度独立的多项目环境。
   - **优点：** 主项目实际上并不直接包含子模块的代码文件，它包含了对子模块特定提交的**引用**（通常是一个指向该提交的Git哈希值）和一个子模块的URL。
   - **缺点：** 管理起来较为复杂，需要使用特定的Git命令来同步和更新子模块。
2. **子树（Subtrees）:** Git子树与子模块类似，但它提供了一种更简单的方式来合并和管理外部项目。子树允许你将另一个项目作为子目录包含在你的Git仓库中，但不需要为外部项目维护一个单独的仓库引用。
   - **适用场景：** 子树适合于需要较少独立性和更紧密集成的环境。
   - **优点：** 管理起来相对简单。
   - **缺点：** 更新外部仓库的代码可能比子模块更复杂。
3. **包管理器（如npm, Maven, pip等）:** 如果你的项目是某种编程语言或框架的一部分，你可以使用相应的包管理器来引入其他项目作为依赖项。例如，在JavaScript项目中，你可以通过npm或yarn引入其他项目；在Java项目中，可以使用Maven或Gradle；在Python项目中，则可以使用pip。
   - **适用场景：** 当你的项目是某种特定编程语言或框架的一部分，且需要引用在常用包管理器上发布的库时。例如，Web开发、Java应用开发等。
   - **优点：** 简化了依赖管理和版本控制；便于自动化构建和部署。
   - **缺点：** 受限于包管理器的支持和依赖库在包管理器上的可用性。
4. **作为依赖库直接复制代码:** 这是一种简单但不太推荐的方法，即直接将其他项目的代码复制到你的项目中。这种方法不利于跟踪外部代码的更改，也可能引起版权或许可问题。
   - **适用场景：** 对于小型或临时项目，或者当你只需要外部项目的一小部分时。
   - **优点：** 简单直接。
   - **缺点：** 难以跟踪外部代码的更新；可能会有许可和版权问题。
5. **使用Git仓库作为依赖:** 在某些语言或框架中，你可以直接在依赖文件中指定Git仓库的URL（例如，在`package.json`或`Gemfile`中）。这允许你直接从Git仓库中安装代码，而不是通过传统的包管理器。
   - **适用场景：** 当你需要的依赖不在任何包管理器上可用，或者你需要直接从源码安装最新版本时。
   - **优点：** 可以直接使用Git仓库的最新代码。
   - **缺点：** 更新和版本控制可能不如包管理器那样方便。

选择哪种方法取决于特定需求，例如项目大小、依赖项的复杂性、更新频率以及希望如何管理这些依赖项。通常，子模块和子树是管理大型、复杂项目依赖的常用选择。而对于依赖特定语言的库，使用包管理器通常是更方便的方法。

#### 更换博客主题

Hexo的默认主题是hexo-theme-landscape，当需要更换博客主题时（以我更换为light主题为例）：

1. 在GitHub上找到喜欢的Hexo主题后，建议fork到自己仓库，便于进行定制化修改，比如我所用的[主题](https://github.com/jackhai9/hexo-theme-light)；

2. ##### 以子模块的方式添加到themes目录下

   `git submodule add git@github.com:jackhai9/hexo-theme-light.git themes/light`

   > 当然也可以以子树的方式 或者 作为依赖库直接复制代码（直接git clone到themes目录下并删除.git目录）。更多依赖管理的方式看[这里](#依赖管理)

3. 修改_config.yml中的theme: light

4. 提交修改：`git add .` 然后 `git commit -m "新增主题xxx"`然后 `git push`

5. 如果你对子模块进行了定制化修改，则需要
   1. cd到子模块目录（比如themes/light目录），提交修改到子模块的远程仓库；
   2. 再cd回主项目目录（比如jackhai9.github.io目录），再提交修改到主项目的远程仓库；

#### 更换仓库或分支

对于GitHub Pages，有两种常见的分支选项来发布你的网站：

  - **`master`或`main`分支**：对于用户或组织的 GitHub Pages 网站（如`<你的GitHub用户名>.github.io`），通常直接使用`master`或`main`分支来发布。
  - **`gh-pages`分支**：对于项目仓库，你可以使用特别的`gh-pages`分支来托管 GitHub Pages 内容。

  GitHub提供了选项来选择哪个分支用于GitHub Pages，在仓库的设置中配置，根据项目需求选择最适合的分支。

#### 添加favicon

把favicon.png（或者favicon.ico，取决于主题代码里用的是什么后缀）放到主题source目录下：`themes/主题/source`目录下。

