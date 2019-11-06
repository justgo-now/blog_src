---
title: 最简便的方法基于Next主题搭建Hexo+Github博客
date: 2019-10-1 21:45:53
tags: [博客, hexo]
---



## 前言

Hexo，反正我第一眼看到就喜欢上了它的简约，[感受一下吧](http://geoffen.github.io/)。
如果你喜欢写作，我觉得你可以试试gitbook或者跟着本文搭建一个属于自己的博客空间（即使你不是IT行业的一员），不再受限于第三方博客地址，当然Hexo搭建的博客也是基于github托管的，但是并不需要你购买域名。
经过两天的探索加爬坑，终于把博客在git上安家了，感谢开源的大哥大姐们，由于并非js开发，所以遇到了很多坑，于是也想整理一篇比较完整的博客。

ps:我选择的主题是[Next](https://github.com/iissnan/hexo-theme-next)

<!-- more -->

## 环境配置

### 安装Node.js

[下载Node.js](https://nodejs.org/download/)
参考地址：[安装Node.js](http://www.w3cschool.cc/nodejs/nodejs-install-setup.html)



### 安装Git

下载地址：<http://git-scm.com/download/>



### 安装Hexo

```
$ cd d:/hexo
$ npm install hexo-cli -g
$ hexo init blog
$ cd blog
$ npm install
$ hexo g # 或者hexo generate
$ hexo s # 或者hexo server，可以在http://localhost:4000/ 查看
```

这里有必要提下Hexo常用的几个命令：

1. hexo generate (hexo g) 生成静态文件，会在当前目录下生成一个新的叫做public的文件夹
2. hexo server (hexo s) 启动本地web服务，用于博客的预览
3. hexo deploy (hexo d) 部署播客到远端（比如github, heroku等平台）

另外还有其他几个常用命令：

```
$ hexo new "postName" #新建文章
$ hexo new page "pageName" #新建页面
```

常用简写

```
$ hexo n == hexo new
$ hexo g == hexo generate
$ hexo s == hexo server
$ hexo d == hexo deploy
```

常用组合

```
$ hexo d -g #生成部署
$ hexo s -g #生成预览
```

现在我们打开<http://localhost:4000/> 已经可以看到一篇内置的blog了。



### Hexo主题设置

这里以主题yilia为例进行说明。

#### 安装主题

```
$ hexo clean
$ git clone https://github.com/litten/hexo-theme-yilia.git themes/yilia
```

#### 启用主题

修改Hexo目录下的_config.yml配置文件中的theme属性，将其设置为yilia。

#### 更新主题

```
$ cd themes/yilia
$ git pull
$ hexo g # 生成
$ hexo s # 启动本地web服务器
```

现在打开<http://localhost:4000/> ，会看到我们已经应用了一个新的主题。



### hexo部署

#### 使用hexo deploy部署

hexo deploy可以部署到很多平台，具体可以[参考这个链接](https://hexo.io/docs/deployment.html). 如果部署到github，需要在配置文件_config.xml中作如下修改：

```
deploy:
  type: git
  repo: git@github.com:geoffen/geoffen.github.io.git
  branch: master
```

然后在命令行中执行

```
hexo d
```

即可完成部署。

注意需要提前安装一个扩展：

```
$ npm install hexo-deployer-git --save
```

### 使用git命令行部署

不幸的是，上述命令虽然简单方便，但是偶尔会有莫名其妙的问题出现，因此，我们也可以追本溯源，使用git命令来完成部署的工作。

#### clone github repo

```
$ cd d:/hexo/blog

$ git clone https://github.com/geoffen/geoffen.github.io.git .deploy/geoffen.github.io
```

将我们之前创建的repo克隆到本地，新建一个目录叫做.deploy用于存放克隆的代码。

#### 创建一个deploy脚本文件

```
hexo generate
cp -R public/* .deploy/geoffen.github.io
cd .deploy/geoffen.github.io
git add .
git commit -m “update”
git push origin master
```

简单解释一下，hexo generate生成public文件夹下的新内容，然后将其拷贝至geoffen.github.io的git目录下，然后使用git commit命令提交代码到geoffen.github.io这个repo的master branch上。

需要部署的时候，执行这段脚本就可以了（比如可以将其保存为deploy.sh）。执行过程中可能需要让你输入Github账户的用户名及密码，按照提示操作即可。



### 添加插件

添加sitemap和feed插件

```
$ npm install hexo-generator-feed
$ npm install hexo-generator-sitemap
```

修改_config.yml，增加以下内容

```
# Extensions
Plugins:
- hexo-generator-feed
- hexo-generator-sitemap

#Feed Atom
feed:
  type: atom
  path: atom.xml
  limit: 20

#sitemap
sitemap:
  path: sitemap.xml
```

配完之后，就可以访问`http://geoffen.github.io/atom.xml`和`http://geoffen.github.io/sitemap.xml`，发现这两个文件已经成功生成了。



### 添加404公益

GitHub Pages有提供制作404页面的指引：[Custom 404 Pages](https://help.github.com/articles/custom-404-pages)。

直接在根目录下创建自己的404.html或者404.md就可以。但是自定义404页面仅对绑定顶级域名的项目才起作用，GitHub默认分配的二级域名是不起作用的，使用hexo server在本机调试也是不起作用的。

推荐使用[腾讯公益404](http://www.qq.com/404/)。



### 添加about页面

```
$ hexo new page "about"
```

之后在sourceaboutindex.md目录下会生成一个index.md文件，打开输入个人信息即可，如果想要添加版权信息，可以在文件末尾添加：

```
<div style="font-size:12px;border-bottom: #ddd 1px solid; BORDER-LEFT: #ddd 1px solid; BACKGROUND: #f6f6f6; HEIGHT: 120px; BORDER-TOP: #ddd 1px solid; BORDER-RIGHT: #ddd 1px solid">
<div style="MARGIN-TOP: 10px; FLOAT: left; MARGIN-LEFT: 5px; MARGIN-RIGHT: 10px">
<IMG alt="" src="https://avatars1.githubusercontent.com/u/168751?v=3&s=140" width=90 height=100>
</div>
<div style="LINE-HEIGHT: 200%; MARGIN-TOP: 10px; COLOR: #000000">
本文链接：<a href="<%= post.link %>"><%= post.title %></a> <br/>
作者： 
<a href="http://geoffen.github.io/">令狐葱</a> <br/>出处： 
<a href="http://geoffen.github.io/">http://geoffen.github.io/</a>
<br/>本文基于<a target="_blank" title="Creative Commons Attribution-ShareAlike 4.0 International (CC BY-SA 4.0)" href="http://creativecommons.org/licenses/by-sa/4.0/"> 知识共享署名-相同方式共享 4.0 </a>
国际许可协议发布，欢迎转载，演绎或用于商业目的，但是必须保留本文的署名 
<a href="http://geoffen.github.io/">令狐葱</a>及链接。
</div>
</div>
```



### 添加Fork me on Github

[获取代码](https://github.com/blog/273-github-ribbons)，选择你喜欢的代码添加到hexo/themes/yilia/layout/layout.ejs的末尾即可，注意要将代码里的you改成你的Github账号名。



### 添加支付宝捐赠按钮及二维码支付

支付宝捐赠按钮在D:/hexo/themes/yilia/layout_widget目录下新建一个zhifubao.ejs文件，内容如下

```
<p class="asidetitle">打赏他</p>
<div>
<form action="https://shenghuo.alipay.com/send/payment/fill.htm" method="POST" target="_blank" accept-charset="GBK">
    <br/>
    <input name="optEmail" type="hidden" value="your 支付宝账号" />
    <input name="payAmount" type="hidden" value="默认捐赠金额(元)" />
    <input id="title" name="title" type="hidden" value="博主，打赏你的！" />
    <input name="memo" type="hidden" value="你Y加油，继续写博客！" />
    <input name="pay" type="image" value="转账" src="http://7xig3q.com1.z0.glb.clouddn.com/alipay-donate-website.png" />
</form>
</div>
```

添加完该文件之后，要在D:/hexo/themes/yilia/_config.yml文件中启用，如下所示，添加zhifubao

```
widgets:
- category
- tag
- links
- tagcloud
- zhifubao
- rss
```

或者

很简单，打开主题配置文件_config.yml
添加字段：

```
reward_comment: 坚持原创技术分享，您的支持将鼓励我继续创作！
alipay: http://ww3.sinaimg.cn/small/937882b5jw1f4dalzfc5nj20p00vu0um.jpg

```

`alipay:` 填写的是支付宝或者微信的收款二维码图片地址。



## 生成部署三部曲

```
hexo clean #清空缓存
hexo g #生成静态网页
hexo d #部署到github
```



## 主题优化



[样式预览](http://chaserr.github.io/)

### Next主题的安装使用

首先从github上clone到本地，在终端cd / 切换到你通过Hexo init生成的根目录，
然后在终端输入命令：

```
$ git clone https://github.com/iissnan/hexo-theme-next themes/next
```

进入站点的全局配置文件：_config.yml
找到theme字段：设置为

```
theme: next
```

到这里可以验证一下主题是否被启用。终端输入：

```
hexo s -- debug
```

然后本地访问[http://localhost:4000，看看效果，在没有部署到github上之前，一般都可以这样在本地进行预览。](http://localhost:4000%EF%BC%8C%E7%9C%8B%E7%9C%8B%E6%95%88%E6%9E%9C%EF%BC%8C%E5%9C%A8%E6%B2%A1%E6%9C%89%E9%83%A8%E7%BD%B2%E5%88%B0github%E4%B8%8A%E4%B9%8B%E5%89%8D%EF%BC%8C%E4%B8%80%E8%88%AC%E9%83%BD%E5%8F%AF%E4%BB%A5%E8%BF%99%E6%A0%B7%E5%9C%A8%E6%9C%AC%E5%9C%B0%E8%BF%9B%E8%A1%8C%E9%A2%84%E8%A7%88%E3%80%82/)

关于站点全局_config.yml配置文件的其他一些参数：

```
# Site
title: 朝夕 #博客名
subtitle: 朝闻道，夕死可以。 #博客副标题
description: #给搜索引擎看的，对站点的描述，可以自定义
author: 童星 #作者名称，显示在网站最底部
language: zh-Hans #语言选择，这里表示中文
email: #你的联系邮箱
timezone:

# URL
## If your site is put in a subdirectory, set url as 'http://yoursite.com/child' and root as '/child/'
url: chaserr.github.io  #可以填写你的站点域名
root: /
permalink: :year/:month/:day/:title/
permalink_defaults:

# Directory
source_dir: source
public_dir: public
tag_dir: tags
archive_dir: archives
category_dir: categories
code_dir: downloads/code
i18n_dir: :lang
skip_render:

# Writing , 设置生成博文的默认格式
new_post_name: :title.md # File name of new posts
default_layout: post #默认布局
titlecase: false # Transform title into titlecase
external_link: true # Open external links in new tab
filename_case: 0 #把文件名称转换为（1）小写或者大写
render_drafts: #显示草稿
post_asset_folder: false  #启动Asset文件夹
relative_link: false #把链接改为与根目录相对位置
future: true #显示未来的文章
highlight: #代码块的设置
  enable: true
  line_number: true
  auto_detect: false
  tab_replace:
```

默认的大致就这些，另外给主题添加相关功能的时候后面会慢慢加上其他一些参数。

#### 给站点添加rss和sitemap功能

打开终端，切换到站点根目录，输入：

```
$ npm install hexo-generator-feed
$ npm install hexo-generator-sitemap
```

在全局配置文件_config.yml进行插件配置：（为防止好找，在最下面进行配置）

```
#插件配置
plugins: hexo-generator-feed 
#- hexo-generator-sitemap 

feed:
  type: atom ##feed类型 atom或者rss2
  path: atom.xml ##feed路径
  limit: 20  ##feed文章最小数量
```

#### 给站点添加本地搜索功能

- 使用swiftype添加站内搜索

去swiftype官网注册一个账号，按照步骤选择自己喜欢的搜索样式，配置完成后，选择install，然后会出现[![Snip20160531_10](http://7xuupy.com1.z0.glb.clouddn.com/Snip20160531_10.png)](http://7xuupy.com1.z0.glb.clouddn.com/Snip20160531_10.png)
如图所示，我不知道为什么，这个代码框无法滚动，所以我们必须将它全部复制出来

```
<script type="text/javascript">
  (function(w,d,t,u,n,s,e){w['SwiftypeObject']=n;w[n]=w[n]||function(){
  (w[n].q=w[n].q||[]).push(arguments);};s=d.createElement(t);
  e=d.getElementsByTagName(t)[0];s.async=1;s.src=u;e.parentNode.insertBefore(s,e);
  })(window,document,'script','//s.swiftypecdn.com/install/v2/st.js','_st');
  
  _st('install','7Qoo1zkKbfSsp5bzUjQu','2.0.0');
</script>
```

然后复制install —-2.0.0之间的一段代码，添加到全局配置文件里：

```
# 搜索插件 hexo-generator-sitemap
swiftype_key: 7Qoo1zkKbfSsp5bzUjQu
```

- 添加本地搜索

直接在全局配置文件添加参数：

```
search:
  path: search.xml
  field: post
```

search.xml这些都是默认带有的

#### 给站点添加友情链接功能

直接在全局配置文件添加参数：

```
links_title: 友情链接
links:
	#百度: http://www.baidu.com/
	#新浪: http://example.com/
```

其实也可以新建一个文件专门放置这些需要链接的网站，这样方便管理，不必每次都对全局配置文件进行更改

> 如果没有链接，那么友情链接不会显示，为了测试，可以随便写一个

#### 给站点添加多说评论、热评、分享功能

直接在全局配置文件添加参数：

```
# Disqus  disqus评论,  与多说类似, 国内一般使用多说
#disqus_shortname: 
duoshuo_shortname: chaser

# 多说热评文章 true 或者 false
duoshuo_hotartical: true

# 多说分享服务
duoshuo_share: true
duoshuo_info:
  ua_enable: true
  admin_enable: false
  user_id:
	  admin_nickname:
```

> **注意：**这里的`duoshuo_shortname`是需要你去多说主页 – [传送门](http://duoshuo.com/) 注册一个账号，然后填写你的多说域名，这里填写的就是你在多说填写的域名
> 例如：我的是：`http://chaser.duoshuo.com/`，那么我填写的就是`chaser`
>
> - 这里配置的是所有页面都支持评论，但是后面有的页面不需要，比如标签页，分类页，关于等等，那么就单独设置。

### Next主题的优化

这之前，先对主题配置文件进行一些配置，打开博客根目录/themes/next，这个路径，然后打开，主题配置文件_config.yml：

#### 配置个人头像

在主题配置文件里找到avatar字段，(如果没有就添加)

```
# Sidebar Avatar
# in theme directory(source/images): /images/avatar.jpg
# in site  directory(source/uploads): /uploads/avatar.jpg
avatar: http://ww4.sinaimg.cn/small/937882b5jw1f4db4lroy9j20hs0npmy6.jpg
```

> 我采用的是地址，并没有将图片放在本地（采用新浪博客相册）

#### Next主题选择

在主题配置文件_config.yml里，找到scheme字段

```
# ---------------------------------------------------------------
# Scheme Settings
# ---------------------------------------------------------------

# Schemes
#scheme: Muse
#scheme: Mist
scheme: Pisces
```

- Muse - 默认 Scheme，这是 NexT 最初的版本，黑白主调，大量留白

- Mist - Muse 的紧凑版本，整洁有序的单栏外观

- Pisces - 双栏 Scheme，小家碧玉似的清新

  我选择的是第三种，去掉

  ```
  #
  ```

  ，然后把第一个前面加上

  ```
  #
  ```

  > 每进行一步都可以自己去进行预览：
  >
  > hexo s –debug

#### Next主题菜单配置

```
# ---------------------------------------------------------------
# Menu Settings
# ---------------------------------------------------------------

# When running the site in a subdirectory (e.g. domain.tld/blog), remove the leading slash (/archives -> archives)
menu:
  home: / #主页
  categories: /categories #分类页面
  about: /about #关于
  archives: /archives #归档
  tags: /tags #标签
  commonweal: /404.html  #公益404


# Enable/Disable menu icons.
# Icon Mapping:
#   Map a menu item to a specific FontAwesome icon name.
#   Key is the name of menu item and value is the name of FontAwsome icon. Key is case-senstive.
#   When an question mask icon presenting up means that the item has no mapping icon.
menu_icons:
  enable: true #是否显示图标
  #KeyMapsToMenuItemKey: NameOfTheIconFromFontAwesome
  home: home
  about: user
  categories: th
  tags: tags
  archives: archive
  commonweal: heartbeat
```

> 1. 如果需要添加你自己自定义的菜单，那么就只需要在menu下添加相关的菜单项，然后新建页面与之匹配，然后去next主题目录的`languages`目录下找到我们之前配置的主题所使用的语言，（我们用的是`zh-Hans`）对其进行国际化。
> 2. menu_icon的设置采用`key-value`。key对应上面的menu里面的菜单项，大小写一致，value就是对应[FontAwsome icon](http://www.bootcss.com/p/font-awesome/#icons-web-app)这个网站的图标的名字，去掉前缀icon
>
> 关于这里的所有的图标都支持从`FontAwsome icon`这个网站获取，当然也可以自己放在资源文件夹里（一般比如个人头像(`avatar`)，网站图标(`favicon`)等)。
> 当然也支持从网址获取，推荐使用`七牛`云存储。当然也可以使用新浪博客的相册，谷歌相册，甚至QQ空间，都可以，只要能获取到图片网址，并且你不会轻易删掉。

##### 添加关于，标签，分类，公益404页面

菜单配置好了，但是我们还得新建一些页面与之相匹配，否则点击进去找不到。这里说一下，对于`标签页`和`分类页`，我们需要在新建一篇文章的时候指定它的标签和分类。对于刚开始建立的博客，点进去可能是空的，所以等会新建一篇文章试试，这之前，先建立相关页面。

###### 添加标签页

打开终端，进入博客根站点，输入：

```
hexo new page tags
```

进入博客根目录/source路径，找到tags文件夹，可以看到生成了index.md文件。可以使用编辑器打开
在里面添加`tags`和`comments`
[![Snip20160531_11](http://7xuupy.com1.z0.glb.clouddn.com/Snip20160601_11.png)](http://7xuupy.com1.z0.glb.clouddn.com/Snip20160601_11.png)

> 1. `tags`设置页面的类型
> 2. `comments`用来控制是否显示评论。

###### 添加分类页

```
hexo new page categories
```

进入博客根目录/source路径，找到categories文件夹，可以看到生成了index.md文件。可以使用编辑器打开
在里面添加`tags`和`comments`
[![Snip20160531_11](http://7xuupy.com1.z0.glb.clouddn.com/Snip20160601_11.png)](http://7xuupy.com1.z0.glb.clouddn.com/Snip20160601_11.png)

> 1. `tags`设置页面的类型
> 2. `comments`用来控制是否显示评论。

###### 添加关于页

```
hexo new page about
```

进入博客根目录/source路径，找到categories文件夹，可以看到生成了index.md文件。可以使用编辑器打开
在里面添加`tags`和`comments`
[![Snip20160531_11](http://7xuupy.com1.z0.glb.clouddn.com/Snip20160601_11.png)](http://7xuupy.com1.z0.glb.clouddn.com/Snip20160601_11.png)

> 1. `tags`设置页面的类型
> 2. `comments`用来控制是否显示评论。

about这个页面一般都是填写一些你的个人信息，要不要无所谓，我这里给about页面添加了一个背景音乐盒.去网易云音乐，创建一个自己的歌单，然后分享，然后找到自己分享的动态，点击链接，[![Snip20160531_12](http://7xuupy.com1.z0.glb.clouddn.com/Snip20160601_12.png)](http://7xuupy.com1.z0.glb.clouddn.com/Snip20160601_12.png)
出现如图的页面，然后点击生成’’外链播放器’’字样。将出现的代码复制，粘贴到你的about页面
[![Snip20160531_13](http://7xuupy.com1.z0.glb.clouddn.com/Snip20160601_13.png)](http://7xuupy.com1.z0.glb.clouddn.com/Snip20160601_13.png)

新建一篇文章给它添加分类和标签试试吧：

```
hexo new "Hexo教程"
```

通过mou编辑器打开：添加`tags`和`categories`

```
---
title: title #文章標題
date: 2016-06-01 23:47:44 #文章生成時間
categories: "Hexo教程" #文章分類目錄 可以省略
tags: #文章標籤 可以省略
	 - 标签1
	 - 标签2
 description: #你對本頁的描述 可以省略
---
```

###### 添加公益404页面

直接在根目录的source路径下，新建一个404.html文件，就可以了
附：404.html

```
<!DOCTYPE HTML>
<html>
<head>
  <meta http-equiv="content-type" content="text/html;charset=utf-8;"/>
  <meta http-equiv="X-UA-Compatible" content="IE=edge,chrome=1" />
  <meta name="robots" content="all" />
  <meta name="robots" content="index,follow"/>
</head>
<body>

<script type="text/javascript" src="http://www.qq.com/404/search_children.js"
        charset="utf-8" homePageUrl="your site url "
        homePageName="回到我的主页">
</script>

</body>
</html>
```

> 提示：`homePageUrl`记得修改成你的博客域名

### Next主题侧边栏社交链接

打开主题配置文件_config.yml：

```
# ---------------------------------------------------------------
# Sidebar Settings
# ---------------------------------------------------------------
# Social Links
social:
	GitHub: https://github.com/chaserr
	新浪微博: http://weibo.com/lovaxiang
	FaceBook: https://www.facebook.com/x.tongxing
	GooglePlus : https://plus.google.com/u/0/117848542581702766384
	豆瓣: https://www.douban.com/people/lovax/

# Social Links Icons
social_icons:
 enable: true #控制是否显示图标
  # Icon Mappings.
  # KeyMapsToSocalItemKey: NameOfTheIconFromFontAwesome
  GitHub: github
  新浪微博: weibo
  FaceBook: facebook
  GooglePlus: google-plus
  豆瓣: globe  #douban
```

> 这里的配置和Menu里一样，
>
> 1. `social`后面跟着的是你的社交网站的主页地址
> 2. `social_icons`是FontAwesome网站的图标名称
>
> 不过豆瓣icon好像是没有的，我在github上issue过，他们给我的解决方案就是去官网找到图标，放到本地资源、。、
> 所以亲们也不用去issue了。直接用别的代替吧，如果真想显示你的豆瓣的话。

### 开启打赏功能

很简单，打开主题配置文件_config.yml
添加字段：

```
reward_comment: 坚持原创技术分享，您的支持将鼓励我继续创作！
alipay: http://ww3.sinaimg.cn/small/937882b5jw1f4dalzfc5nj20p00vu0um.jpg
```

> `alipay:` 填写的是支付宝或者微信的收款二维码图片地址。

### 给文章添加阅读量

#### 配置LeanCloud

打开[LeanCloud](https://leancloud.cn/login.html#/signin)官网，注册一个账号，完成激活
[![Snip20160531_14](http://7xuupy.com1.z0.glb.clouddn.com/Snip20160601_14.png)](http://7xuupy.com1.z0.glb.clouddn.com/Snip20160601_14.png)
点击创建新应用之后，进入新创建的应用
[![Snip20160531_15](http://7xuupy.com1.z0.glb.clouddn.com/Snip20160601_15.png)](http://7xuupy.com1.z0.glb.clouddn.com/Snip20160601_15.png)
点击创建Class，类名叫做`Counter`。

#### 修改主题配置文件_config.yml

打开主题配置文件，找到`leancloud_visitors`字段或者创建

```
# Show number of visitors to each article.
# You can visit https://leancloud.cn get AppID and AppKey.
leancloud_visitors:
  enable: true
  app_id: ytnok33cvEchgidigtb0WumC-gzGzoHsz #<AppID>
  app_key: SrcG8cy1VhONurWBoEBGGHML #<AppKEY>
```

> 1. `app_id`
> 2. `app_key`
>    [![Snip20160531_18](http://7xuupy.com1.z0.glb.clouddn.com/Snip20160601_18.png)](http://7xuupy.com1.z0.glb.clouddn.com/Snip20160601_18.png)
>
> 将对应key的value复制填写即可。

#### 添加lean-analytics.swing

在主题目录下的\layout_scripts路径下，新建名为：`lean-analytics.swing`的文件，
这里贴上`lean-analytics.swing`代码

```
{% if theme.leancloud_visitors.enable %}

  {# custom analytics part create by xiamo #}
  <script src="https://cdn1.lncld.net/static/js/av-core-mini-0.6.1.js"></script>
  <script>AV.initialize("{{theme.leancloud_visitors.app_id}}", "{{theme.leancloud_visitors.app_key}}");</script>
  <script>
    function showTime(Counter) {
      var query = new AV.Query(Counter);
      var entries = [];
      var $visitors = $(".leancloud_visitors");

      $visitors.each(function () {
        entries.push( $(this).attr("id").trim() );
      });

      query.containedIn('url', entries);
      query.find()
        .done(function (results) {
          var COUNT_CONTAINER_REF = '.leancloud-visitors-count';

          if (results.length === 0) {
            $visitors.find(COUNT_CONTAINER_REF).text(0);
            return;
          }

          for (var i = 0; i < results.length; i++) {
            var item = results[i];
            var url = item.get('url');
            var time = item.get('time');
            var element = document.getElementById(url);

            $(element).find(COUNT_CONTAINER_REF).text(time);
          }
        })
        .fail(function (object, error) {
          console.log("Error: " + error.code + " " + error.message);
        });
    }

    function addCount(Counter) {
      var $visitors = $(".leancloud_visitors");
      var url = $visitors.attr('id').trim();
      var title = $visitors.attr('data-flag-title').trim();
      var query = new AV.Query(Counter);

      query.equalTo("url", url);
      query.find({
        success: function(results) {
          if (results.length > 0) {
            var counter = results[0];
            counter.fetchWhenSave(true);
            counter.increment("time");
            counter.save(null, {
              success: function(counter) {
                var $element = $(document.getElementById(url));
                $element.find('.leancloud-visitors-count').text(counter.get('time'));
              },
              error: function(counter, error) {
                console.log('Failed to save Visitor num, with error message: ' + error.message);
              }
            });
          } else {
            var newcounter = new Counter();
            /* Set ACL */
            var acl = new AV.ACL();
            acl.setPublicReadAccess(true);
            acl.setPublicWriteAccess(true);
            newcounter.setACL(acl);
            /* End Set ACL */
            newcounter.set("title", title);
            newcounter.set("url", url);
            newcounter.set("time", 1);
            newcounter.save(null, {
              success: function(newcounter) {
                var $element = $(document.getElementById(url));
                $element.find('.leancloud-visitors-count').text(newcounter.get('time'));
              },
              error: function(newcounter, error) {
                console.log('Failed to create');
              }
            });
          }
        },
        error: function(error) {
          console.log('Error:' + error.code + " " + error.message);
        }
      });
    }

    $(function() {
      var Counter = AV.Object.extend("Counter");
      if ($('.leancloud_visitors').length == 1) {
        addCount(Counter);
      } else if ($('.post-title-link').length > 1) {
        showTime(Counter);
      }
    });
  </script>

{% endif %}
```

> 网上也有人贴出来这部分代码，但是会出现阅读量出现重复情况，这里的代码是经过测试后出现的，准确无误。

#### 修改post.swing文件

打开主题目录\layout_macro路径,找到post.swig文件，找到`LeanCould PageView #`字样,去掉 其中一个`&nbsp; | &nbsp`，否则部署之后会出现双`||`。

#### 修改layout.swing文件

打开主题目录\layout路径，找到_layout.swing
搜索`</body>`标签，在其上方添加代码：

```
{% if theme.leancloud_visitors.enable %}
{% include '_scripts/lean-analytics.swig' %}
{% endif %}
```

### 给页面添加High一下

打开`博客根目录\themes\next\layout_partials\header.swig`，在`<ul> ... /ul>`标签之间加入以下代码：

```
<li> <a title="把这个链接拖到你的Chrome收藏夹工具栏中" href='javascript:(function() {
    function c() {
        var e = document.createElement("link");
        e.setAttribute("type", "text/css");
        e.setAttribute("rel", "stylesheet");
        e.setAttribute("href", f);
        e.setAttribute("class", l);
        document.body.appendChild(e)
    }
 
    function h() {
        var e = document.getElementsByClassName(l);
        for (var t = 0; t < e.length; t++) {
            document.body.removeChild(e[t])
        }
    }
 
    function p() {
        var e = document.createElement("div");
        e.setAttribute("class", a);
        document.body.appendChild(e);
        setTimeout(function() {
            document.body.removeChild(e)
        }, 100)
    }
 
    function d(e) {
        return {
            height : e.offsetHeight,
            width : e.offsetWidth
        }
    }
 
    function v(i) {
        var s = d(i);
        return s.height > e && s.height < n && s.width > t && s.width < r
    }
 
    function m(e) {
        var t = e;
        var n = 0;
        while (!!t) {
            n += t.offsetTop;
            t = t.offsetParent
        }
        return n
    }
 
    function g() {
        var e = document.documentElement;
        if (!!window.innerWidth) {
            return window.innerHeight
        } else if (e && !isNaN(e.clientHeight)) {
            return e.clientHeight
        }
        return 0
    }
 
    function y() {
        if (window.pageYOffset) {
            return window.pageYOffset
        }
        return Math.max(document.documentElement.scrollTop, document.body.scrollTop)
    }
 
    function E(e) {
        var t = m(e);
        return t >= w && t <= b + w
    }
 
    function S() {
        var e = document.createElement("audio");
        e.setAttribute("class", l);
        e.src = i;
        e.loop = false;
        e.addEventListener("canplay", function() {
            setTimeout(function() {
                x(k)
            }, 500);
            setTimeout(function() {
                N();
                p();
                for (var e = 0; e < O.length; e++) {
                    T(O[e])
                }
            }, 15500)
        }, true);
        e.addEventListener("ended", function() {
            N();
            h()
        }, true);
        e.innerHTML = " <p>If you are reading this, it is because your browser does not support the audio element. We recommend that you get a new browser.</p> <p>";
        document.body.appendChild(e);
        e.play()
    }
 
    function x(e) {
        e.className += " " + s + " " + o
    }
 
    function T(e) {
        e.className += " " + s + " " + u[Math.floor(Math.random() * u.length)]
    }
 
    function N() {
        var e = document.getElementsByClassName(s);
        var t = new RegExp("\\b" + s + "\\b");
        for (var n = 0; n < e.length; ) {
            e[n].className = e[n].className.replace(t, "")
        }
    }
 
    var e = 30;
    var t = 30;
    var n = 350;
    var r = 350;
    var i = "//7xuupy.com1.z0.glb.clouddn.com/tongxingSibel%20-%20Im%20Sorry.mp3";
    var s = "mw-harlem_shake_me";
    var o = "im_first";
    var u = ["im_drunk", "im_baked", "im_trippin", "im_blown"];
    var a = "mw-strobe_light";
    var f = "//s3.amazonaws.com/moovweb-marketing/playground/harlem-shake-style.css";
    var l = "mw_added_css";
    var b = g();
    var w = y();
    var C = document.getElementsByTagName("*");
    var k = null;
    for (var L = 0; L < C.length; L++) {
        var A = C[L];
        if (v(A)) {
            if (E(A)) {
                k = A;
                break
            }
        }
    }
    if (A === null) {
        console.warn("Could not find a node of the right size. Please try a different page.");
        return
    }
    c();
    S();
    var O = [];
    for (var L = 0; L < C.length; L++) {
        var A = C[L];
        if (v(A)) {
            O.push(A)
        }
    }
	})()    '>High一下</a> </li>
```

> `//7xuupy.com1.z0.glb.clouddn.com/tongxingSibel%20-%20Im%20Sorry.mp3"`可以替换成任意你想要的音乐地址

### 设置网站的图标Favicon

favicon图标也就是我们打开一个网页，出现在最浅的图标样式，可以自定义，首先我们需要一个favicon.ico的图标，可以去[比特虫](http://www.bitbug.net/)网站制作，网上的做法是将其放在根目录的source文件夹里。然后在主题目录的layout_partials路径下，修改header.swig的meta标签，我实验了一下，并不能配置成功呢，所以代码就不贴了，这里介绍我的做法:
图表制作好后，上传到云存储空间，获取图片的网址，然后打开主题配置文件_config.yml，找到favicon字段，将图片网址粘贴在后面，即可。

```
favicon: http://ww4.sinaimg.cn/square/937882b5jw1f4db4lroy9j20hs0npmy6.jpg #网站图标
```

到此，Next的主题算是简单的美化了一下，你现在可以在本地预览，也可以将其部署到github上去哦。

### 增加评论功能

Hexo Next 集成 utterances 评论系统

#### GitHub 配置与脚本获取

1. 创建存放 comments 的代码仓库，必须为 public，且可创建 issue。
2. [install utterances app](https://github.com/apps/utterances) 点击这个链接安装utterances app到刚刚创建的那个仓库。
3. 打开 <https://utteranc.es/> ，根据提示的步骤生成所需的js script。也可在下面代码的基础上更新自己的信息。

```
<script src="https://utteranc.es/client.js"
        repo="maoqyhz/comments" # onwer/repo
        issue-term="pathname"   # 命名issue的格式，默认pathname
        label="Comment"         # 创建issue的tag，默认Comment
        theme="github-light"    # 评论系统theme，github-light或github-dark
        crossorigin="anonymous" # 跨域，默认
        async>
</script>
```

#### Hexo Next 主题配置

首先，进入主题目录，在 `layout/_third-party/comments/` 中创建 `utterances.swig`，并添加以下代码：

```
{% if theme.utterances.enable %}
<script src="https://utteranc.es/client.js"
        repo="{{ theme.utterances.repo }}"
        issue-term="{{ theme.utterances.issue_term }}"
        label="Comment"
        theme="{{ theme.utterances.theme }}"
        crossorigin="anonymous"
        async>
</script>
{% endif %}
```

再在 `layout/_partials/comments.swig` 最后添加以下代码：

```
{% if theme.utterances.enable %}
 <div class="comments" id="comments">
    {% include '../_third-party/comments/utterances.swig' %}
 </div>
{% endif %}
```

最后，在主题配置文件 `theme/_config.yml` 中添加如下配置：

```
# utterances 
utterances:
  enable: true
  repo:                  # owner/repo
  issue_term: pathname   # pathname, url, title
  theme: github-light    # github-light or github-dark
```

重新生成网页，就能看到评论系统了。



### 添加分享功能

多说已经不可用

参考`https://github.com/theme-next/hexo-next-share`



## 问题解决

### 标签页和分类页无内容显示

首先创建两个页面

```
$ hexo new page categories
$ hexo new page tags
```



打开 `categories` 和`tags` 文件夹下的 `index.md` ，在最下面一行加一行文字就行，注意中间有空格。

```txt
type: categories
```



### 生成静态页面报错

```
FATAL Something's wrong. Maybe you can find the solution here: http://hexo.io/docs/troubleshooting.html
Template render error: (unknown path) [Line 7, Column 23]
  Error: Unable to call `the return value of (posts["first"])["updated"]["toISOString"]`
```

解决方案

```
把

plugins:
- hexo-generator-feed
改成

plugins:
  hexo-generator-feed
就行了，把短横线去掉
```

