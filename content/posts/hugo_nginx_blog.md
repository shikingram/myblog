---
title: "hugo + nginx 搭建博客记录"           
author: "Kingram"              
date: 2021-07-13            
lastmod: 2021-07-13       

tags: [              
    "blog",
    "nginx",
]
---

<p>作为一个萌新Gopher，经常逛网站能看到那种极简的博客，引入眼帘的不是花里胡哨的图片和样式，而是黑白搭配，简简单单的文章标题，这种风格很吸引我。正好看到煎鱼佬也在用这种风格的博客，于是卸载了我的wordpress开始抄袭，o(*￣︶￣*)o</p>

## Hugo简介

`Hugo`是由Go语言实现的静态网站生成器。简单、易用、高效、易扩展、快速部署。

## 准备和环境

我直接在我的阿里云服务器上使用hugo了，环境如下：

- 操作系统：公共镜像CentOS 8.1 64位
- Nginx版本：Nginx 1.16.1
- 域名：kingram.top
- 阿里云服务器：8.136.204.132

## Hugo使用

### 安装Hugo

由于我用的是`Linux`系统，直接`wget`下载安装就好了，其他系统参考[Hugo中文文档](https://www.gohugo.org/)其他版本参考[Hugo Releases](https://github.com/spf13/hugo/releases)

```shell
cd ~
wget https://github.com/gohugoio/hugo/releases/download/v0.85.0/hugo_0.85.0_Linux-64bit.tar.gz
tar -zxvf hugo_0.85.0_Linux-64bit.tar.gz
mv hugo /usr/bin/
```

检查是否安装成功

```
hugo version
```

输出

```
hugo v0.85.0-724D5DB5 linux/amd64 BuildDate=2021-07-05T10:46:28Z VendorInfo=gohugoio
```

就说明安装成功啦

### 创建blog站点

在当前目录执行命令创建blog站点

```
hugo new site myblog
```

这个`myblog`就是项目的名字了，创建的目录如下

```
├── archetypes
│   └── default.md
├── config.toml         # 博客站点的配置文件
├── content             # 博客文章所在目录
├── data                
├── layouts             # 网站布局
├── static              # 一些静态内容
└── themes              # 博客主题
```

我们的博客文章就放在`content`目录下的`posts`中，只需要按照Markdown格式编写，hugo就会读取到文章然后展示在博客中。

### 安装主题

不用主题是无法使用hugo的，我这里用的是`hermit`主题，开发者是[Track3](https://ojbk.im/)，然后我魔改了下，抄袭了煎鱼佬的样式，改为了黑白结合的。

安装依次执行以下命令：

```
cd myblog 
# clone到
git clone https://github.com/Track3/hermit.git ./themes/hermit
```

### 使用主题

将`hermit`主题中`exampleSite`目录下的内容拷贝到当前目录`myblog`下

```
cp themes/hermit/exampleSite/* ./ -r
```

可以通过修改`config.toml`文件来更改配置

贴上我的`config.toml`文件配置，是抄了煎鱼佬(#^.^#)

```
baseURL = "http://kingram.top"
languageCode = "zh-hans"
defaultContentLanguage = "en"
title = "Kingram"
theme = "hermit"
# enableGitInfo = true
pygmentsCodefences  = true
pygmentsUseClasses  = true
# hasCJKLanguage = true  # If Chinese/Japanese/Korean is your main content language, enable this to make wordCount works right.
rssLimit = 10  # Maximum number of items in the RSS feed.
copyright = "This work is licensed under a Creative Commons Attribution-NonCommercial 4.0 International License." # This message is only used by the RSS template.
enableEmoji = true  # Shorthand emojis in content files - https://gohugo.io/functions/emojify/
googleAnalytics = "UA-166045776-1"
# disqusShortname = "yourdiscussshortname"

[author]
  name = "Kingram"

[blackfriday]
  # hrefTargetBlank = true
  # noreferrerLinks = true
  # nofollowLinks = true

[taxonomies]
  tag = "tags"
  # Categories are disabled by default.

[params]
  since = "2018"
  toc = true

 customCSS = ["css/a.css"]
  
  dateform        = "Jan 2, 2006"
  dateformShort   = "Jan 2"
  dateformNum     = "2006-01-02"
  dateformNumTime = "2006-01-02 15:04 -0700"

  # Metadata mostly used in document's head
  # description = ""
  # images = [""]

  # homeSubtitle = "Coding, Thinking, Life"
  footerCopyright = ' &#183; <a href="http://beian.miit.gov.cn/">苏ICP备2021012652号</a>'
  # bgImg = ""  # Homepage background-image URL

  # Prefix of link to the git commit detail page. GitInfo must be enabled.
  # gitUrl = "https://github.com/username/repository/commit/"

  # Toggling this option needs to rebuild SCSS, requires Hugo extended version
  justifyContent = false  # Set "text-align: justify" to `.content`.

  relatedPosts = false  # Add a related content section to all single posts page

  code_copy_button = true # Turn on/off the code-copy-button for code-fields
  
  # Add custom css
  # customCSS = ["css/foo.css", "css/bar.css"]

  # Social Icons
  # Check https://github.com/Track3/hermit#social-icons for more info.

  [[params.social]]
    name = "github"
    url = "https://github.com/k1ngram4"

[menu]

  [[menu.main]]
    name = "文章"
    url = "posts/"
    weight = 10
	
  [[menu.main]]
    name = "标签"
    url = "tags/"
    weight = 10

  [[menu.main]]
    name = "关于"
    url = "about/"
    weight = 20

```

设置好配置文件后在`myblog`目录下执行`hugo`命令即可生成`public`文件夹，这个文件夹就是我们站点的根目录文件夹，后面nginx中部署时指定的根目录也是这个。如果想使用`github pages`只要将这个目录放在`github`托管，每次改完提交即可。

```
hugo
```

## 使用Nginx部署

### 安装Nginx

```
dnf -y install http://nginx.org/packages/centos/8/x86_64/RPMS/nginx-1.16.1-1.el8.ngx.x86_64.rpm
```

执行命令查看nginx版本

```
nginx -v
```

### 配置Nginx

修改nginx配置文件的用户，否则后面会出现权限问题

```
vi /etc/nginx/nginx.conf
```

按i进入编辑模式

修改`user nginx;`为`user  root;`

按下Esc键，并输入`:wq`保存退出文件。

修改配置文件

```
cd /etc/nginx/conf.d
vi default.conf
```

按i进入编辑模式

在`location`大括号内，修改以下内容。

```
location / {
    #将该路径替换为您的网站根目录。
    root   /root/myblog/public/;
    #添加默认首页信息
    index  index.html index.htm;
}
```

按下Esc键，并输入`:wq`保存退出文件。

### 启动Nginx

运行以下命令启动Nginx服务。

```
systemctl start nginx
```

运行以下命令设置Nginx服务开机自启动。

```
systemctl enable nginx
```

这样服务端Nginx就配置完成了，要想通过kingram.top访问，还需要关闭防火墙（或者配置端口）、配置dns解析、配置阿里云端口安全组（80端口）等操作，我是直接关了防火墙

### 关闭防火墙

永久关闭防火墙：

```
systemctl disable firewalld
```

运行`systemctl status firewalld`命令查看当前防火墙的状态

```
● firewalld.service - firewalld - dynamic firewall daemon
   Loaded: loaded (/usr/lib/systemd/system/firewalld.service; disabled; vendor preset: enabled)
   Active: inactive (dead)
     Docs: man:firewalld(1)

```

dns解析和配置阿里云安全组可以通过阿里云文档配置，这样就可以使用域名直接访问我们的博客了。(๑′ᴗ‵๑)Ｉ Lᵒᵛᵉᵧₒᵤ❤



后面准备搭个自动部署的脚本，等搞完了再写个文章总结下。┏(＾0＾)┛

