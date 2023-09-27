---
title: Github Pages + Hexo NexT搭建个人博客
date: 2023-04-17 22:39:07
tags:
- 经验
- Github Pages
- Hexo
- NexT主题
categories:
- 博客搭建
comments: true
---

某日心血来潮，想着要给无所事事的自己找一些事做，耗费了数个安静的夜晚，我的个人博客诞生了 ：)

在这个过程中，遇到了一些问题，也有过许多思量。因此在博客基本建完之后，回过头来，总结一番，记录下这几天的收获，也可供有幸来此的友友参考。

<!-- more -->

## 心理博弈篇

对于个人博客形式的考虑，我还是比较明确的，坚定地选择了**Github Pages**，排除了CSDN、简书等站点。

选择Github Pages原因在于：

- 想要折腾一番，若是坚持不下来，那坚持写博客就更是空谈了
- 免费，新手上路，甚是友好
- 有蛮多开源且好看的主题的说

好吧，好看的主题确实很多，但也让我纠结了很久。最开始选择的静态博客框架是Jekyll，这个也是Github官方支持的框架，符合心意的主题如[HuxPro](https://github.com/Huxpro/huxpro.github.io)、[TeXt](https://github.com/kitian616/jekyll-TeXt-theme)等。后来又发现了Hexo框架和**[NexT](https://github.com/theme-next/hexo-theme-next)主题**，我觉得很可，黑白配色，简约到我心里。

对于Markdown编辑器，也有过一番纠结。鉴于平时只用过有道云笔记&飞书文档写东西，我又在网上大查一通，力求选一个简单使用方便快捷易上手的编辑器。过程不表，最终摆在面前的有两种方案：

- VSCode + Markdown插件
- Typora

听说Typora现在是个付费软件，本想投入VSCode的怀抱。又听说Beta版本还是可以免费使用的，于是又打算尝试一下。当然，可能小部分原因是因为之前听朋友夸过叭。

最终，一番思量下来，博客架构也算是基本确定了：**Github Pages + Hexo NexT + Typora**

## 一知半解篇

### Github Pages

#### 前置条件

创建Github Pages，只需要两个条件：

- Github账号
- 本地安装Git

##### Github相关

已有账号的友友可以默默思考一下自己的用户名，是否中意；没有账号的友友自行注册时记得想一个酷酷的name，因为个人Github Pages默认的域名便是``username.github.io``，且不支持自定义。So，如果如我一般，不满足现有的用户名，可以自行修改。修改过程文字描述如下：

1. Github单击头像
2. 点击**Settings**
3. 左侧列表单击**Account**
4. 选择**Change username**
5. 之后大概是一些注意事项，详细阅读了解，修改即可

此外，还推荐友友们检查一下Repository default branch，如果默认是main，那么建议修改为master。此操作涉及到后续主题配置及部署。检查默认分支步骤文字描述如下：

1. 进入Settings界面
2. 左侧列表点击Repositories
3. 映入眼帘的便是了

##### Git相关

后续主题配置和博客编写都会在本地进行，因此需要使用Git进行代码管理。

此处简述一下我在clone repo时的曲折，以供参考。

Github比较常用的clone方式有两种：

- https
- ssh

**https**方式在clone时需要验证密码，**ssh**需要提前上传公钥。于是图省事选择了**https**。结果···

在clone代码时，有时会直接报错，如下

> fatal: unable to access 'https://github.com/treeknows/treeknows.github.io.git/': OpenSSL SSL_read: Connection was reset, errno 10054

有时会弹出来**github**的认证窗口，验证之后又弹出了**Openssh**相关的窗口，但总是验证失败。

基于上述的报错，google了蛮多资料，涉及到Git配置修改、github access token等许多内容，但没能真正解决问题。

由此改用更为熟悉的**ssh**方式，需要在本地先生成公钥，复制并上传。对此处内容陌生的友友可以自行google学习，只需要理解现在在做的是什么，带着目的去查找教程，就不会盲目且迷糊了。

> 即：需要使用ssh的方式从github clone代码，所以需要提前上传本地的公钥，那么就需要本地有一个公钥。那么如何检查本地是否有公钥呢？如何在本地生成/重新生成公钥呢？有了公钥具体要怎么上传到github呢？
>
> 清楚目的，带着问题，google即可。
>
> 此方法亦适用于本篇其他没有详述过程的步骤，乃至日常工作、学习。

#### Pages创建

Github Pages的创建过程是比较简单的，过程文字描述如下：

1. 进入个人主页界面
2. 点击头像左边的➕
3. 点击new repository
4. 进入创建界面**Create a new repository**
   1. Repository name必须填写``username.github.io``，如``zhangsan.github.io``
   2. Description描述，可写可不写
   3. 公开与否建议设为**Public**，涉及到后续评论系统
   4. README file & .gitignore file & license，自由选择
5. 配置OK后点击**Create repository**按钮
6. 此时应该就到仓库的主页了
7. 点击**仓库主页的Settings**，区别于之前的Settings
8. 点击左边列表中的**Pages**
9. 界面中Source选择**Depoly from a branch**
10. Branch分别选择**master** & **/(root)**，没有修改默认分支为master的应该选择main
11. 此时博客网站应该已经可以访问了，在浏览器地址栏输入``username.github.io``

此时的网站还属于比较简陋的状态，下一步，**搞个主题**。

> 如果如果此时网站打不开，可能是需要等待一下。在仓库主页的右下，有一个**Environment**，可以查看当前github-pages state
>
> 如果如果还是不行，可能是需要一个README文件，或一个简单的index.html。请勇敢解决它。

### Hexo NexT

本站使用的是基于Hexo框架的NexT主题，黑白色调，主打一个简约。主题包含四种布局（暂且称为布局），简单理解就是上下结构和左右结构，可以根据个人喜好选择。

#### 前置条件

**Hexo**是基于**NodeJS**编写的，所以需要安装**NodeJS**和**npm**工具。

##### NodeJS安装

首先需要在官网下载安装包，[NodeJS官网](http://nodejs.cn/download/)，安装包下载完成之后运行。除了安装路径建议手动修改一下，其他保持默认一路Next即可。

>macOS or Linux安装方式请自行google

安装完成之后，打开**git bash**，输入以下命令检查安装是否成功

```
$ node -v
v18.15.0

$ npm -v
9.5.0
```

一般情况下是不会有问题的，我安装的很顺利，所以没有相关的报错可以提供解决思路。

##### Hexo安装

安装Hexo只需执行下述命令，亦可以在[Hexo官网](https://hexo.io/zh-cn/)查看安装及使用方法。

```
$ npm install -g hexo-cli

# 查看hexo版本
$ hexo -v
```

我当时在安装hexo结束之后，查看版本会报错，找不到hexo命令。

似乎看起来是安装没有成功，但是**npm install**却也没有打印出任何error信息，最终的解决办法为：

```
$ npx hexo -v
hexo-cli: 4.3.0
os: win32 10.0.19044
node: 18.15.0
v8: 10.2.154.26-node.25
···

```

是的，执行`npx hexo`而不是`hexo`

> 2023.09.28更新：
>
> Windows下想要直接执行`hexo`需要手动添加环境变量。
>
> 相对路径为：BLOG_DIR\hexo\node_modules\.bin

#### 主题安装

万事俱备，只欠安装。

但是在安装之前，要唠叨两句。考虑到博客的框架、主题、配置是在本地环境下的，如果涉及到更换电脑，便需要推倒重来，于是便新建了**hexo**分支来管理博客的主题和配置文件，博客站点的真正内容会部署到**master**分支。这样如果换了电脑，便只需要安装**NodeJS**和**Hexo**，就能直接得到原本的博客环境啦。

##### Hexo初始化

首先是要先把新建的仓库clone到本地，新建博客源码分支**hexo**，初始化hexo，具体命令如下：

```

$ git clone xxxxxx // xxxxx是github上复制的clone地址

$ cd xxxxxx // clone到本地会生成一个跟仓库同名的目录

$ git checkout -b hexo // 创建并切换到hexo分支

$ hexo init hexo // 初始化hexo，第一个hexo是cmd，第二个hexo是新建hexo目录

$ cd hexo // 进入新建的hexo目录

$ npm install // 安装相关依赖

$ hexo g // generate，生成静态网站

$ hexo s // server，本地预览
```

到这里就可以打开浏览器输入**localhost:4000**，进行本地预览了。

> 上述步骤中，如有需要**hexo**请灵活替换为**npx hexo**，以及过程来自于回忆，如果哪一步的输出有报错，请google解决一下，也欢迎**留言咨询**。

此时本地预览的界面是Hexo默认的主题，接下来介绍安装NexT主题。

##### NexT主题安装

```
$ cd hexo //确保当前目录处于hexo目录下

// clone NexT主题，如果使用https clone报错，可以自行替换为ssh的链接
$ git clone https://github.com/theme-next/hexo-theme-next themes/next 
```

此时NexT主题就已经安装到本地了，其配置保存在**themes/next**目录下。若想要启用NexT主题，还需要修改Hexo的配置文件`_config.yml`，注意此文件应位于**hexo**目录下，称为**站点配置文件**。在**themes/next**路径下也有一个`_config.yml`，称之为**主题配置文件**。

在**站点配置文件**中搜索**theme**，修改为如下值：

```
# Extensions
## Plugins: https://hexo.io/plugins/
## Themes: https://hexo.io/themes/
# 启用主题
theme: next
```

执行如下命令本地验证主题是否生效：

```
$ hexo clean // 清理之前生成的资源
$ hexo g // 重新生成
$ hexo s // 启动本地服务
```

打开浏览器输入**localhost:4000**，进行本地预览。

##### 部署到Github Pages

本地预览无误，便可部署到Github Pages了。

首先需要安装插件**hexo-deployer-git**

```
$ npm install hexo-deployer-git --save
```

修改**站点配置文件**，搜索**deploy**

```
# Deployment
## Docs: https://hexo.io/docs/one-command-deployment
deploy:
  type: git
  repo:
  	# 需要替换为自己仓库的链接，根据https或ssh方式不同自由选择，我已经上传了公钥，所以选择ssh
    github: git@github.com:treeknows/treeknows.github.io.git
  # 此处似乎只能选择master，配置为main时，实际还是会部署到master分支
  branch: master
```

可以将配置修改为如上，记得**替换自己的仓库链接**。此外配置branch时，设置为**main**实测是无效的，依旧会push到master分支，如果没有master分支，则会自己创建。这也是为什么在前面推荐**修改默认创建分支为master**，来自强迫症患者的倔强。

修改配置后，需要手动部署到Github

```
$ hexo g -d
```

部署成功后，稍待片刻，便可访问自己的博客查看效果了。

##### 备份Hexo

此时博客已然部署完毕，但是相关的配置还只存在于本地，需要push到Github。

```
$ git branch // 请确保在hexo分支

$ git add . // 添加文件

$ git commit // 提交

$ git push --set-upstream origin hexo // 推送到远端
```

推送到Github之后，还需要在Github上将**hexo**设为默认分支，设置过程文字描述如下：

1. 打开博客仓库主页
2. 点击仓库主页的Settings
3. 在**General**页面，设置Default branch为**hexo**

此时博客配置就已经备份成功了，如果切换了电脑，只需要：

1. clone 仓库 (默认分支为hexo)

   ```
   $ git clone git@github.com:*.github.io.git
   ```

1. 在本地 *.github.io 文件夹下通过 Git bash 依次执行下列指令：

   ```
   $ npm install hexo
   $ npm install 
   $ npm install hexo-deployer-git
   ```

> 切记，不需要 hexo init 这条指令

此时便可以在新电脑上愉快的writing了。

> 实测过程中遇到一个问题：在B电脑上clone下来博客的配置后，修改了一些配置，执行`npm install`安装了一个插件，并将这些修改的部分提交到了Github。在A电脑同步最新配置后，本地`hexo g`会报错，怀疑可能的原因有：
>
> - 我添加了ignore文件，可能导致B电脑安装的插件被忽略，没有push到服务器，导致A电脑配置了插件，却没有真正下载插件
> - 也可能是需要先执行`hexo clean`，再重新生成
>
> 最终的解决是上述两点都做了，在A电脑手动安装了插件，clean之后再生成预览。

### Typora

Markdown编辑器使用的是Typora 0.11.18 beta版本，下载地址[点击此处](https://ghpym.lanzoui.com/b00zng7gd)，下载之后安装，运行。此时运行会有弹框提示升级，不升级便只能退出应用，无法使用。

此时需要退出应用，按照以下步骤修改注册表即可正常使用。

1. 按`Win+R`打开运行窗口，输入`regedit`，点确定，打开注册表，依次展开`计算机\HKEY_CURRENT_USER\Software\Typora`，然后在`Typora`上右键，点`权限`，选中`Administrtors`，把权限全部设置为`拒绝`。
2. 打开Typora，此时应该可以正常打开应用了。还需要在偏好设置里把自动更新关掉。

做完上面两步，Typora beta版本应该就能正常使用了。看网上的说法，最新版本也有破解的办法，只是我个人觉得没必要。

另外提一下正版的Typora购买也就不到100米，可以在三台设备上使用，可以一直使用，不排除大版本升级后需要重新购买的可能。所以比较推荐的做法是，先感受一下Typora，如果使用感受不错以及新版本有心动的功能的话，可以考虑支持一下正版。

初体验，确实不错，插入图片相关的功能还在探索ing，再写几篇博客就能背下来Markdown的语法了。

### 个性化主题

到此，博客的现状应该是NexT主题，但是还没有修改任何配置，因此接下来就要对配置进行一些修改，大致会分为：

- 基础配置
- 个性配置

基础配置诸如网站标题、描述一类的配置，篇幅不会太长；个性配置便有很多说头了，我会另写一篇博客专门介绍，不会太久。

基础配置位于站点配置文件的开头，大致有以下几种：

```
# Site
title: // 网站标题
subtitle: // 子标题，在主页显示，看起来的效果更像一个签名
description: // 在侧边栏显示，效果也像是签名
keywords: // 网站关键字
author: // 作者，会显示在侧边栏以及网站底部
language: zh-CN // 语言
timezone: 'Asia/Shanghai' // 时区

url: // 填写博客站点的地址即可
```

基础配置大致如此，更多的配置含义及细节，随后更新。

## 豁然开朗篇

这篇博客陆陆续续用了三个工作日的安静夜晚才完成，导致写到此处时竟忘记了豁然开朗在哪里，忘记了原本想表述在此的内容，可见

- 写博客并非一件容易之事，还是需要花费一定的时间和精力
- 写出完整内容之前可以先列个大纲或者写个备忘，把想说的先简单记录

也如上所说，本文是在本站配置好之后才开始写，且并非连续写完，难免存在疏漏，以及逻辑、语句不通之处，欢迎指正。

也因为图床目前还在研究，所以有意的选择了无图模式，表述不够清晰之处，欢迎留言交流。

最后，希望这一段时间的努力不会浪费，我能够坚持把博客写下去，给自己一个沉淀、成长的机会。

：）
