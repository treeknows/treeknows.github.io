---
title: Github + PicGo + Typora搭建图床
date: 2023-04-28 01:20:52
tags:
- PicGo
- 图床搭建
- 经验
categories:
- 博客搭建
comments: true
---

前面两篇博客介绍了个人博客搭建的过程，接下来就可以写博客了。但是写博客不可避免的需要用到一些图片，所以拥有一个图床是非常重要的事情。

网上有一些免费的图床，但是存在挂掉的风险。也有一些付费的云，如七牛、阿里云等，但是比较麻烦，需要花时间去研究怎么使用，可能在传图片的时候也需要手动操作许多步骤。

我个人比较倾向于使用Github作为图床，主打一个免费+稳定。然后在Typora中发现了**PicGo**这个应用，可以简化上传的动作。所以决定采用**GitHub + PicGo + Typora**搭建一个图床，方便日后博客使用图片。

<!-- more -->

## Github

首先需要在GitHub上创建repo，权限为需要设置为`public`，用于存放图片。具体过程不表。

## PicGo

### 下载

下载[PicGo v2.3.1](https://github.com/Molunerfinn/PicGo/releases/tag/v2.3.1)，这是最新的release版本，选择合适的格式下载，下载完成后安装。

### 配置

打开PicGo，图床设置选择GitHub，有三个必须的配置

- 仓库名：填写为`GitHub_username/repo_name`
- 分支名：填写分支名，一般新建的repo默认分支为`master`或`main`，可以在repo的settings界面查看
- Token：需要手动创建
  - 打开Settings
  - 左侧菜单最下面点击`Developer settings`
  - 选择`Personal access tokens`下的`Tokens(classic)`
  - 点击右侧`Generate new token`按钮，选择classic
  - 将生成的token复制到PicGo的设置里

保存配置之后就可以上传图片了。打开上传区，选择一张图片，拖住上传。上传成功后会将URL复制到剪贴板。登陆Github看到图片已经成功上传了。

此时图片上传到了repo的根目录，如果想要上传至指定的目录，可以在PicGo设置界面**设定存储路径**，如设置为`imgs/`，图片便会上传至repo下的imgs目录。注意一定要加`/`，否则变得是图片名字，而不是图片存储的路径。

还有一个配置是**自定义域名**，这个在一开始配置时是不需要动的。如果后续需要使用CDN加快图片访问速度，可以去修改

参考设置如图：

![PicGo_settings](https://raw.githubusercontent.com/treeknows/blog_pic/master/imgs/PicGo_settings.jpg)

## Typora

Typora支持了PicGo上传图片，也就是意味着只要复制粘贴图片到Typora，就可以自动使用PicGo上传图片并生成链接，十分方便。

配置过程为：

- Typora左上角一次点击**文件->偏好设置->图像**
- 如下图所示，依次设置：
  - 图片存储方式：上传图片
  - PicGo路径：选择为PicGo的安装路径
  - 点击**验证图片上传选项**

![image-20230507204623740](https://raw.githubusercontent.com/treeknows/blog_pic/master/imgs/image-20230507204623740.png)

Typora会自动验证上传，一般情况下都会成功的。不需要额外配置什么。

此时就可以直接复制图片到Typora了。

但是会有一个问题，图片上传成功，但是无法在Typora预览。

从网上随便找了一张图片，复制链接到Typora，发现是可以正常预览的。那么可以定位问题出在域名上。

~~搜索了一下相关问题，在[一篇博客](http://www.duheweb.com/post/20210421125522.html)中找到了一种解释：~~

> ~~**使用Typora**~~
> ~~上面的设置完成后，在 Typora 里插入图片时会把图片复制到本地的指定目录，待编辑完md文件后再选择格式 -> 图像 ->上传所有本地图片，就可以自动把md文件里的图片上传到github，并自动把图像的URL更换。不设置为插入图片就自动上传是因为上传到github后，自定义的图片URL生效需要一段时间，URL没生效时在Typora里无法预览图片。~~
>
> ~~注意：typora 在线预览github图片失败是因为自定义的URL还不能访问，而typora没有自动刷新远程图片的功能，稍等一下重启md文件，就能看到typora把github图片加载出来了。~~

~~但是个人认为有一些出入的地方，复制图片的链接到浏览器的地址栏，是可以正常打开图片，看起来不像是URL无法访问的问题。~~

~~同时在GitHub上找了一些类似链接的图片，同样无法预览，所以有可能是**无法访问GitHub**的原因，可能需要挂VPN在测试一下。~~

> ~~不巧这台电脑没有VPN客户端，只能后面再试了。~~

~~尽管在博客中的解释并不能让我信服，但是它提供的一种思路是可以借鉴的：~~

- ~~文件->偏好设置->图像中，**插入图片时**不要设置为上传图片，而设置为复制到指定路径，然后在本地建立一个文件夹存放图片，此时插入图片是可以正常预览的。然后在编辑完成之后，点击**格式->图像->上传所有本地图片**，将本地图片统一上传。~~

~~另一种可能的解决方式就是，PicGo中配置自定义域名，使用CDN加速，可能能解决这个问题。~~

**解决**：修改hosts文件

原因是github屏蔽掉了图片，添加以下内容到本地的hosts文件

```
185.199.108.133 raw.githubusercontent.com
185.199.109.133 raw.githubusercontent.com
185.199.110.133 raw.githubusercontent.com
185.199.111.133 raw.githubusercontent.com
```

重启Typora即可。

参考自：[博客](https://blog.csdn.net/Bilal_0/article/details/126078013)

## 参考

本文只是简略记录了一下过程，详细的可以参考以下：

[PicGo官方教程](https://picgo.github.io/PicGo-Doc/zh/guide/config.html#github%E5%9B%BE%E5%BA%8A)

[知乎上的一个教程](https://zhuanlan.zhihu.com/p/489236769)
