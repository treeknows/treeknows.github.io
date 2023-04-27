---
title: 基于Github Pages + NexT主题的个性化配置
date: 2023-04-21 00:22:50
tags:
- Github Pages
- Hexo
- NexT主题
- 经验
categories:
- 博客搭建
comments: true
---



>本文详细介绍了Github Pages + Hexo框架 + NexT主题的个性化配置，涉及到网站基本信息、评论系统、Google/Bing收录等内容。



## 前言

书接上文，站点部署完成之后，剩下的重中之重便是网站的配置了。诸如网站基本信息之类的配置，在第一篇博客已经简单介绍，这些配置往往在部署完成之后就会修改，不然无法体现出作者本人的资料，也就谈不上部署完成。剩下的便是很多个性化的配置，这里才是本人以及多数人重点关注的内容，所以单开一篇，记录自己DIY的过程以及一点点心得。

<!-- more -->

>既面向大众，尽微薄之力；也记录下来，和记忆妥协。（时间久了真的会忘···）

首先重新明确一个前提，使用的是Hexo框架+NexT主题，在本地站点相关的目录下会有两个配置文件`_config.yml`，其中一个位于init hexo框架的目录下，称之为**站点配置文件**；另一个位于框架目录下的*themes/next/*目录下，称之为**主题配置文件**。相比之下，**站点配置文件**在外层目录，主题配置文件在内层目录，希望这个描述可以清晰你对配置文件的不同之处，避免混淆。

## 配置博客

### 总体原则

配置博客的一个**总体原则就是了解配置文件中各项配置的具体含义，基于自己的理解修改成想要的配置，并善用hexo的本地服务进行调试，灵活修改**。总的来说，是一件耗时且枯燥的工作，所以直接搜索相关的配置博客是一个很好的选择。

当然，如我这般，搜到的博客都是若干年前发表的，NexT版本的不一致导致很多配置都不能直接适用，所以我只好尽可能的了解各项配置的含义，慢慢完善成自己想要的效果。

> 如果你如我一般，请参考上面的思路，慢慢配置。

### 具体配置

- 取消网站底部的强力驱动字样

  - 在**主题配置文件**中搜索powered，设为false即可

    ```
      # Powered by Hexo & NexT
      ## 是否显示网站底部的强力驱动字样
      powered: false
    ```

    

- 增加本地搜索功能

  - 安装插件，执行 `npm install hexo-generator-searchdb --save`
  - 修改**站点配置文件**，在其最下方新增如下配置

  ```
  search:
  	path: search.xml
  	field: post
  	format: html
  	limit: 10000
  ```

  - 修改**主题配置文件**，启用搜索功能

    ```
    # Local Search
    # Dependencies: https://github.com/theme-next/hexo-generator-searchdb
    local_search:
      ## 启用
      enable: true
      ## 设为auto，输入关键词之后自动显示列表
      # If auto, trigger search by changing input.
      ## 设为manual，输入关键词之后回车显示列表
      # If manual, trigger search by pressing enter key or search button.
      trigger: auto
      # Show top n results per article, show all results by setting to -1
      top_n_per_article: -1
      # Unescape html strings to the readable one.
      unescape: false
      # Preload the search data when the page loads.
      ## 预加载，打开可能会影响加载速度
      preload: false
    ```

    > 题外话：似乎在博客数量上去之后，本地搜索的性能会受到影响，NexT提供了一个三方的搜索服务`Algolia Search`，但是初期似乎没必要安排上。

- 设置侧边栏头像

  - 编辑**主题配置文件**，搜索字段`avatar`

    ```
    ## 设置侧边栏头像
    avatar:
      # Replace the default image and set the url here.
      ## 选择自己喜欢的图片，保存在相对路径hexo/themes/next/source/images下即可
      url: /images/avatar.gif
      # If true, the avatar will be dispalyed in circle.
      ## 设为true，头像会变成圆形，默认是方框
      rounded: false
      # If true, the avatar will be rotated with the cursor.
      ## 设为true，鼠标触碰到时头像框会旋转
      rotated: false
    
    ```

- 设置网站图标（浏览器标签页显示的小图标）

  - 在**主题配置文件**中搜索`favicon`字段

    ```
    favicon:
      ## 主要是这两个，不同尺寸
      small: /images/favicon-16x16-next.png
      medium: /images/favicon-32x32-next.png
      ## 这两个也做了替换，但没有验证效果
      apple_touch_icon: /images/apple-touch-icon-next.png
      safari_pinned_tab: /images/logo.svg
      #android_manifest: /images/manifest.json
      #ms_browserconfig: /images/browserconfig.xml
    ```
    
    可以看到有不同类型的图标，可以从网站下载或制作自己喜欢的图标，替换对应的源文件即可。（推荐一个图标素材站点：[iconfont](https://www.iconfont.cn/)）

- 修改文章底部的标签样式，#改为图标

  - 在**主题配置文件**中搜索`tag_icon`，将其置为true即可

- 设置建站时间

  - 在**主题配置文件**中搜索`since`，取消注释修改其值即可

    ```
    footer:
      # Specify the date when the site was setup. If not defined, current year will be used.
      since: 2023
      ···
    ```

- 首页截断设置

  - 首推的方法是在文章合适的位置手动添加`<!-- more -->`，这样就可以生成截断效果了

  - 在**主题配置文件**中，搜索excerpt_description，实际效果未经过测试

    ```
    # Automatically excerpt description in homepage as preamble text.
    ## 首页截断设置，本地实测如果文章中添加了<!-- more -->，这个设为false也不会影响截断效果
    excerpt_description: true
    
    # Read more button
    # If true, the read more button will be displayed in excerpt section.
    read_more_btn: true
    ```

- 增加代码复制按钮

  - **主题配置文件**中搜索copy_button，设置为true即可

    ```
    codeblock:
      # Code Highlight theme
      # Available values: normal | night | night eighties | night blue | night bright | solarized | solarized dark | galactic
      # See: https://github.com/chriskempson/tomorrow-theme
      ## 选择代码风格
      highlight_theme: normal
      # Add copy button on codeblock
      copy_button:
        enable: true
        # Show text copy result.
        ## 显示复制结果
        show_result: true
        # Available values: default | flat | mac
        ## 风格，mac风格好像挺奇怪的
        style:
    ```

  - 这里顺便也贴一下代码风格的设置方法。如上中注释*highlight_theme*

- 增加网站访客人数和阅读量

  - 在**主题配置文件**中搜索busuanzi_count，将其启用

    ```
    # Show Views / Visitors of the website / page with busuanzi.
    # Get more information on http://ibruce.info/2015/04/04/busuanzi
    busuanzi_count:
      enable: true
      ## 总访客以及图标
      total_visitors: true
      total_visitors_icon: fa fa-user
      ## 总访问次数及图标
      total_views: true
      total_views_icon: fa fa-eye
      ## 文章阅读量及图标
      post_views: true
      post_views_icon: fa fa-eye
    ```
    
    另外，本地验证时发现数字很抽象，部署之后查看会正常。
    
    >另外，目前busuanzi还无法重置访客和访问次数

- 永久链接格式

  - 在**站点配置文件**中搜索permalink

    ```
    permalink: :title/
    permalink_defaults: :year/:month/:day/:title/
    ```

    默认文章的链接是年+月+日+标题的，可以修改成只有标题，似乎这样可以优化[SEO](https://www.google.com.hk/search?q=seo%E6%98%AF%E4%BB%80%E4%B9%88&oq=seo&aqs=chrome.1.69i57j0i512j46i340i512l2j69i60l2j69i65j69i61.2595j0j7&sourceid=chrome&ie=UTF-8)

- 设置页面加载时顶部进度条

  - 在主题配置文件中搜索pace，enable设为true，在theme处选择具体的风格

    ```
    pace:
      enable: true
      # Themes list:
      # big-counter | bounce | barber-shop | center-atom | center-circle | center-radar | center-simple
      # corner-indicator | fill-left | flat-top | flash | loading-bar | mac-osx | material | minimal
      theme: minimal
    ```

  - 下载插件，参考[pace官方](https://github.com/theme-next/theme-next-pace)执行以下命令：

    ```
    $ cd themes/next
    $ ls
    _config.yml  crowdin.yml  docs  gulpfile.js  languages  layout  LICENSE.md  package.json  README.md  scripts  source
    
    $ git clone https://github.com/theme-next/theme-next-pace source/lib/pace
    ```

- **Google收录博客网站**

  > 一个比较关键的配置。由于依旧是无图模式，所以只能尽可能的用文字描述清楚。且由于时间过的比较久，也无法完全重新跑一次流程，所以有些地方还是有点模糊的。如有问题，**欢迎留言**。
  >
  > 另外做完这些操作之后，google收录还是会需要两三天的时间，所以再心急也只能安稳等着。

  - 查看是否已被收录，在地址栏输入：`site:http://xxxx.github.io`，如果能显示出自己站点的网页，说明已收录，反之则需要手动提交。

    > 一般新站是不会被收录的，如果不主动提交，理论上有足够多的高质量外链，是有可能被Google主动收录的。

  - 进入Google Web Master [Search Console](https://www.google.com/webmasters/tools/home?hl=zh-CN) 并登录，一般都是谷歌账号直接登录的

  - 登陆之后找到添加资源的地方，资源类型选择第二种：**网址前缀**

  - 输入自己的地址之后点击继续，会弹出一个验证所有权的窗口。**切记不要关闭网页**。

  - 验证方法有很多种，这里推荐选择第二种：**HTML标记**

  - 此方法会给出一串HTML代码，例如：

    ```
    <meta name="google-site-verification" content="rMF1JqbmMsHu2M1zSLQ482HWMOd8u-4z-zWViLWMXHg" />
    // NOTE: 这是我随机输入的网址生成的标记，请务必使用自己站点生成的代码
    ```

  - 此时有两种方法：

    - 如果是NexT主题，在**主题配置文件**中搜索`google_site_verification`，并将HTML代码content的内容填上去。

      ```
      // 举例！
      # Google Webmaster tools verification.
      # See: https://www.google.com/webmasters
      google_site_verification:rMF1JqbmMsHu2M1zSLQ482HWMOd8u-4z-zWViLWMXHg
      ```

    - 其他情况，可以手动将改代码添加至swig文件中，打开*themes/next/layout/_partials/head/head.swig* 文件，在其中添加生成的代码。我添加在了第二行，如下所示：

      ```
      <meta charset="UTF-8">
      <meta name="google-site-verification" content="rMF1JqbmMsHu2M1zSLQ482HWMOd8u-4z-zWViLWMXHg" />
      <meta name="viewport" content="width=device-width, initial-scale=1, maximum-scale=2">
      ```

  - 添加完成之后，需要执行`hexo clean && hexo g -d`部署

  - 部署完成之后打开博客网站，鼠标右键查看网页源代码，此时在源代码中应该可以搜到我们添加的代码的。

    > 如果没搜到，可以考虑稍作等待，多刷新几次页面，重新查看源代码；或检查上述步骤

  - 成功搜到之后，回到验证所有权的网页，点击验证。

  - 验证成功之后应该就会跳转到控制台的主页啦

- 添加站点地图

  添加完谷歌收录之后，紧接着就要上传网站的站点地图。

  - 首先是需要安装插件，在终端执行cmd

    ```
    npm install hexo-generator-sitemap --save
    ```

  - 在**站点配置文件**中，添加如下配置。我是在文件末尾添加的。

    ```
    # 站点地图
    sitemap:
        path: sitemap.xml
    ```

  - 执行`hexo clean && hexo g`重新生成后就可以在`public`目录下看到`sitemap.xml`文件了。

  - 回到`Google Search Console`，在左侧的菜单中选择站点地图，输入我们生成的站点地图`sitemap.xml`即可

    >默认是已经有博客的地址了的，也就是已经显示了**https://treeknows.github.io/**，我们只需要在输入框中输入`sitemap.xml`并点击提交即可。
    >
    >NOTE：刚添加完站点地图，下面的状态显示的可能不是**成功**，同样需要等待数天，才会更新状态。

- 谷歌分析

  > Google Analytics（GA）挺高级的，需要一定的学习成本。如果对站点SEO感兴趣，可以多多学习以下

  - 首先要创建账号，前往[Google 分析](https://analytics.google.com/)创建账号及媒体资源。

  - 没记错的话，用谷歌账号是可以直接登陆的

  - 登陆之后创建账号，账号名称没有特别的要求

  - 所有都创建完成之后会生成一个`衡量 ID`，格式如`G-8W35JPL36V`

  - 在**主题配置文件**中，搜索`google_analytics`，并填写上述的ID

    ```
    # Google Analytics
    google_analytics:
      // !!!此处填写
      tracking_id: G-8W35JPL36V
      # By default, NexT will load an external gtag.js script on your site.
      # If you only need the pageview feature, set the following option to true to get a better performance.
      only_pageview: false
    ```

  - 重新生成部署之后应该就可以啦

- Bing收录

  Bing收录的过程类似于Google收录，甚至如果Google收录已经做完，Bing可以直接使用google账号登陆，直接导入Google Search Console的内容，非常方便

  - 点击[Bing Webmaster Tools](https://www.bing.com/webmasters/home)开始提交站点
  - 忘记是要先登录还是直接可以添加站点了，总之找到添加站点的地方选择**从 GSC 导入你的网站**，之后可能需要验证以下google账号之类的，然后就一键导入了
  - 记得看下站点地图需不需要手动添加

  >时间久了导致描述比较模糊，遇到问题可以留言。

- 百度收录

  如果没有自己注册域名的话，百度是不可能收录Github Pages的，因为Github把百度的爬虫禁用了，所以如果希望百度可以搜索到自己的博客站点，需要购买域名并和Github Pages绑定一下，详细步骤自行Google。

  对于我来说，添加一个google就足够了，毕竟行业相关，科学上网是难不倒大家的。

- 创建关于我、分类页面

  - 在**主题配置文件**中搜索`menu`可以设置站点首页显示的菜单，如下

    ```
    menu:
      home: / || fa fa-home
      #tags: /tags/ || fa fa-tags
      archives: /archives/ || fa fa-archive
      categories: /categories/ || fa fa-th
      about: /about/ || fa fa-user
      #schedule: /schedule/ || fa fa-calendar
      #sitemap: /sitemap.xml || fa fa-sitemap
      #commonweal: /404/ || fa fa-heartbeat
    ```

    我这边打开了**分类**和**关于**两个界面，此时本地验证的话，可以发现首页已经有相关的菜单栏，但是点进去会报`not found`，需要我们手动创建相关的页面。

  - 在hexo目录下，打开终端，执行以下命令

    ```
    hexo new page categories
    hexo new page about
    ```

    执行之后，会在*hexo/source/*目录下生成两个新的目录*about*和*categories*，两个目录下都会生成一个`index.md`文件，这个便是我们的页面了

  - *categories*目录下的`index.md`只需要修改类型即可，其内容会根据我们每篇博客中设定的分类自动生成

    ```
    ---
    #title: Blog Categories
    date: 2023-04-14 00:36:03
    layout: categories
    type: categories
    comments: false
    ---
    ```

  - *about*目录下的`index.md`除了需要修改类型，还需要我们手动添加内容，就像写一篇博客一样。

    ```
    ---
    title: About Me
    date: 2023-04-14 00:35:50
    layout: about
    type: about
    comments: false
    ---
    
    CONTENT
    ······
    ```

- 添加评论系统 `Gitalk`

  - 登录 **Github** ，右键头像，在下拉菜单中，选择“**Settings**”选项

  - 在左侧菜单选择“**Developer settings**”选项，进入开发者页面

  - 选择 **OAuth Apps** ，并点击“**New OAuth App**”创建新授权应用

  - 设置该应用相关信息，其中

    - Application name为应用名称，如`博客评论`
    - Homepage URL为博客主页，需要和自己的博客一致
    - Authorization callback URL 授权回调页面（同 Homepage URL）

  - 点击创建应用，进入新的页面

  - 在新页面的**Clients secrets**处，点击`Generate a new client secret`，可能需要验证密码才能生成。

  - 保存生成的client secret和 client id，后续要用到

  - 在**主题配置文件**中搜索`gitalk`，配置如下：

    ```
    # Gitalk
    # For more information: https://gitalk.github.io, https://github.com/gitalk/gitalk
    gitalk:
      // 启用
      enable: true
      // github用户名
      github_id: treeknows
      // 在哪个仓库下，推荐博客所在的仓库
      repo: treeknows.github.io
      // 刚刚保存的ID
      client_id: 22fb59ff85c3aef2677a
      // 刚刚生成的secret
      client_secret: 246142feb132313b08ee3ff55678c88244b67b97
      // Github用户名
      admin_user: treeknows
      ## 点击评论框时会不会变黑，说实话true了不太好看
      distraction_free_mode: false # Facebook-like distraction free mode
      # Gitalk's display language depends on user's browser or system environment
      # If you want everyone visiting your site to see a uniform language, you can set a force language value
      # Available values: en | es-ES | fr | ru | zh-CN | zh-TW
      language:
    
    ```

    参考注释进行配置。选择`repo`时要注意，`repo`应该是public的，否则评论是无法使用的。这也就是为什么在第一篇博客中，推荐将博客所在的仓库设为public而不是private，就是为了此时的方便。

  - 在**主题配置文件**中，搜索`comments`，激活gitalk

    ```
    # Multiple Comment System Support
    comments:
      # Available values: tabs | buttons
      style: tabs
      # Choose a comment system to be displayed by default.
      # Available values: changyan | disqus | disqusjs | gitalk | livere | valine
      // !!! 选择gitalk
      active: gitalk
    ```

  - 执行`hexo clean && hexo g -d`重新生成并部署，打开博客选择一篇文章查看效果。

  - 注意，第一次使用评论系统是需要进行一个初始化的，似乎只需要验证一下github账号即可，比较简单。在之后的post了新文章后，自测是不需要再次手动初始化的。（但是网上说依旧需要手动初始化😶

- 增加版权信息

  - 在**主题配置文件**中搜索`creative_commons`

    ```
    creative_commons:
      license: by-nc-sa
      ## 在侧边栏显示一个协议的图标
      sidebar: false
      ## 文章底部显示版权信息
      post: true
      language:
    ```

    将`post`设置为true，重新生成即可。

- 图床选择

  - 把图床放在最后是因为，我还没有熟悉图床的使用。从这两篇无图模式的博客也可以看出来。因此接下来会写一篇关于**图床的使用**的博客，迫使自己尽快掌握图床的使用🤡

## 总结

到这里，配置相关的介绍基本就结束了，可能会有一些没有涉及到的小修改，可以参考我博客源码中的配置文件：[站点配置文件](https://github.com/treeknows/treeknows.github.io/blob/hexo/hexo/_config.yml) & [主题配置文件](https://github.com/treeknows/treeknows.github.io/blob/hexo/hexo/themes/next/_config.yml)，里面会有一部分的中文注释，帮助理解。

另外，如果觉得我的博客风格还不错，又不想浪费过多的时间配置，似乎可以直接clone我的[博客源码](https://github.com/treeknows/treeknows.github.io.git)，修改一些关键信息为自己的信息，实现极速建站。

大致需要的步骤可能为：

1. Fork我的仓库
2. 修改仓库名字为自己的`username.github.io`
3. clone到本地
4. 修改**站点配置文件**中部署相关的的配置，搜索`deploy`
5. 修改基本信息，如网站title、头像之类的
6. 删除我为数不多的博客
7. GA & Gitalk等换成自己的id
8. 提交自己新的站点到Google Search Console
9. 尝试部署

只脑补到这些，未经验证，不包成功哈🙄
