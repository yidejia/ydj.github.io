---
title: Hexo + GitHub 搭建属于个人或团队的技术博客
date: 2016-11-02 17:23:55
tags:
- Hexo
- GitHub
- Blog 博客
categories:
- 移动端

---

现在很多个人或团队都有写博客的习惯，有的会选择将博客发表到如 CSDN、简书等比较受欢迎的地方，毕竟那里聚集了从菜鸟到大神的各类人才，来学习的和来吐槽的都有。本人就是喜欢热闹点的地方😬。不过今天不是介绍怎样将博文发表到这些地方，而是介绍如何使用 [Hexo](https://hexo.io/) + [GitHub](https://github.com) 来搭建一个属于个人或团队的技术博客（从某种程度上也能提升逼格）。当然写博客最重要的还是抱着一种技术沉淀和积累的心态，将自己的所见、所闻、所感分享给大家。对于团队来说，也在一定程度上向外界传递当前团队的技术研发方向，招纳贤士，吸引人才嘛。

虽然这类文章现在满大街都是，但我觉得还是有必要将这两天的搭建过程给记下来，这其中包含了一些细节需要注意的。以下操作都是基于 Mac 系统，其他平台的请另行查阅。同时建议即将动手实战的朋友先通读本文，有些细节可能需要你提前注意的。最后，请自配翻墙工具。

### 一、准备环境

Hexo 的安装需要依赖 Node.js，而对博客的管理我们选择 Git，所以我们要提前准备这两个环境。

#### 安装 Git

使用 brew 来安装 Git：

 `brew install git`

#### 安装 Node.js

首先我们需要通过 cURL 来安装 [nvm](https://github.com/creationix/nvm)：

`$ curl -o- https://raw.githubusercontent.com/creationix/nvm/v0.32.1/install.sh | bash
`

安装完成后，重新启动终端然后执行如下命令来安装 Node.js：

`nvm install stable`

#### 安装 Hexo
经过上述步骤后，最后使用 npm 来安装 Hexo：

`npm install -g hexo-cli`。

### 二、搭建博客
既然我们使用 Hexo 来搭建博客，那势必要熟悉一下 Hexo 的几个常用语法：

1、首先选择一个目录，如/Users/zxq/Document/Blog 作为我们存储博客文件的目录，命令行切换到该目录下，然后执行：

`hexo init`

2、初始化完成后，我们还需要执行下面命令来安装一些依赖：

`npm install`

现在你的目录结构看起来应该是这样的：

![](https://cloud.githubusercontent.com/assets/7321351/19918231/81b8d62a-a104-11e6-8720-318784661abb.png)

因为我后面还装了一些插件，所以可能会多出 node_modules 这个目录，db.json 就是一个数据文件，部署运行后会自动生成。

到这里，博客的基础框架就搭建完成了，接下来我们就来部署一下，看看最终的显示效果。**如果没特别提示的，下面的命令都在本地博客的根目录下操作**。依次执行: `hexo clean`、`hexo generate`、`hexo server`。当运行 `hexo server` 后，命令行中会提示你访问 [http://localhost:4000](http://localhost:4000)，打开浏览器看是否能正常访问。如果失败则需要在命令行窗口中查看失败日志。

简单说一下上面三个命令具体是执行哪些任务：

1. **hexo clean:** 清除 hexo 生成的静态文件，由于 hexo 是一个博客框架，会将你编写的 Markdown 文件转换成相应的 HTML 文件，执行这个命令就可以删除系统为你生成的这些静态文件。
2. **hexo generate:** 通过上面的命令你也许能猜到这个命令的用意了，就是帮你生成静态文件的。
3. **hexo server:** 本地部署，这样你就能直接在浏览器上通过 [http://localhost:4000](http://localhost:4000) 访问你的博客了。

其实上面命令还有对应的快捷方式，这个留给各位朋友自己摸索了。


### 三、编写博文

博客搭建完后，接下来就要靠各位日后的辛勤耕耘和乐于分享了。
我们可以直接通过 `hexo new Hello_World` 命令来创建博文，这种方式默认会将博文放置到博客根目录下的 `source/_post` 目录中。你也可以选择到该目录下创建 Markdown 文件。当编写好博文后，通过上面描述的三条命令 **clean**，**generate**，**server** 重新部署一下就可以了。

当然，编写博文的方式还远远不止这些， 详情请查看 [Hexo 官方文档](https://hexo.io/zh-cn/docs/index.html)。

### 四、GitHub 远程部署博客
完成上面三个步骤后，你现在的博客仅仅是部署在你本地电脑上，只允许同一局域网进行访问，当然如果你有自己的服务器那就另当别论了。下面将介绍如何将你的博客部署到全球最大的程序员交友网站 [GitHub](https://github.com)。在这里我先申明一下，你既然都看到了这篇文章，那至少你对 GitHub 有所了解了，起码也得有个账号吧。

接下来我们登录 GitHub，新建一个仓库。注意，这个仓库名称必须是：GitHub 的账户昵称 + ".github.io"，如

![](https://cloud.githubusercontent.com/assets/7321351/19919154/ce5cfa54-a10b-11e6-8dc7-414ede4ae44b.png)

因为我已经创建过了，所以会提示仓库已存在。你要注意的就是这个仓库名称格式，这个仓库名称格式，这个仓库名称格式，重要事情说三遍。只有这种格式的仓库 GitHub 才能识别。

这里要唠叨两句，很重要的，不要走神。在 GitHub 上每创建一个仓库，那么该仓库默认会指定 master 分支作为其主要分支。博客仓库（xxx.github.io）也是一样的，GitHub 会默认部署 master 分支上的静态文件（前面说过静态文件是 hexo 将博客源文件转化后的产物）。你若不小删除了这些静态文件我们还可以通过源文件再生成，但如果你不小心删除了博客源文件，那可能就麻烦了。所以我们要借助 Git 来管理我们整个博客的源文件了。可能有些人会想再创建一个新的仓库来管理这些源文件，当然这样也没有错，但还有另一种方式，那就是在博客仓库中再创建一个分支，如下图：

![](http://7xqitw.com1.z0.glb.clouddn.com/blog-resgit_hub_new_branch.png)

`注意：hexo 生成的静态文件和博客源文件最好不要混在一起管理，这也是为什么需要两个分支的原因，一个管理静态文件，一个管理博客源文件。`

因为我们大多数时间是花在写博客上，所以选择用 hexo 分支来管理我们的博客源文件，而 master 分支用于管理静态文件。在 hexo 分支上的操作跟普通操作 git 一样，至于怎么更新 master 分支上的静态文件后面会揭晓。

不过现在博客仓库的默认分支还是 master，这里我们要将 hexo 修改为默认分支，方便Git 今后操作源文件。操作如下：

![](https://cloud.githubusercontent.com/assets/7321351/19921082/454918ae-a117-11e6-8163-47707a23d4f4.png)

当执行完上述流程后，打开本地博客根目录下的 **_config.xml** 配置文件，索引 *Deployment* 然后替换为下面的信息：

``` java
# Deployment
## Docs: https://hexo.io/docs/deployment.html
deploy:
  type: git
  repository: https://github.com/xxx/xxx.github.io (你的博客仓库地址）
  branch: master
```

保存后接着执行 `npm install hexo-deployer-git --save`，只有执行该命令我们才能将待会生成的静态文件推送到远程仓库中。

好了，该准备的也准备得差不多了。提醒一下，推送静态文件到远程 master 分支不再是什么 git push，而是这一系列的命令：**hexo clean，hexo generate，hexo deploy**。细心的朋友可能发现最后一个命令不再是上面提到的 **hexo server**，而是**hexo deploy**，相当于将本地的这些静态文件部署推送到远程的 master 分支上。这一切的前提是上面配置文件中的仓库地址要填写正确。

现在就可以到 GitHub 网站上查看博客仓库的 master 分支上是否有那些静态文件，诸如 html，js 和 css 等等。如果正常的话，那么过一会你就能通过 https://xxx.github.io（比如 [https://anenn.github.io](https://anenn.github.io)） 这个地址直接访问你的博客，因为 GitHub 已经为你完成部署工作了。

### 四、博客源文件管理
完成上述流程后，那接下来的也就小菜一碟了。我们要用 Git 来管理我们的文件，基本流程就是：

* 命令行切换到本地博客根目录下，执行 git init，表明我们要用 Git 来管理该目录下的所有文件；
* 然后将本地仓库跟远程仓库进行绑定，执行 git remote add origin xxx(远程仓库地址，如 https://github.com/anenn/anenn.github.io)；
* 切换分支到 hexo，执行 git checkout -b hexo；
* 将源文件先添加到本地仓库中，在此之前我建议先执行一下 hexo clean，将系统为我们生成的静态文件删除，因为它不属于我们的博客源文件，然后再执行 git add . | git commit -m "xxx"；
* 最后就是同步了，执行 git pull | git push

记住，今后一切一切的操作都在 hexo 分支上执行，不要再跑去 master 分支上瞎闹，搞不好会搞挂博客的。

### 五、总结
上面流程下来不出意外的话应该是可以正常运行的，如果有什么问题的请留言反馈。最后想对搭团队博客的朋友说，如果团队成员有写博客的习惯，若让他们每个人在自己电脑上都搞这一套，不用想，肯定不会怎么受欢迎的。所以，可以接受的方式就是让他们 clone 下团队博客的源代码，然后让他们切换到 hexo 分支上去编写博文，最后同步上去再由自动化脚本去执行部署操作。对于他们来说，整个部署流程透明化，他们仅需关心的就是如何写好每一篇博文。

### 参考文献：

1. [Hexo 官方文档](https://hexo.io/zh-cn/docs/index.html)
2. [Hexo利用Github分支在不同电脑上写博客](http://www.dxjia.cn/2016/01/27/hexo-write-everywhere/)
3. [在不同的电脑维护Hexo和写作](http://www.rvclient.com/2016/05/21/hexo-everywhere/)