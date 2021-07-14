---
title: "使用webhook实现git push后自动部署"           
author: "Kingram"              
date: 2021-7-14 22:59           
lastmod: 2021-7-14 23:11     

tags: [              
    "blog",
]       
---

<p>在使用hugo + nginx搭建好博客后，文章可以通过ftp上传到服务器，然后在服务器上再编译成网页，或者本地搭建的hugo环境，编译好网页再上传到服务器，这样做虽然也可以，但是很麻烦，如果每次都这么发布文章，肯定玩几次就不想弄了。</p>
<p>使用webhook就能实现自动部署，其实原理很简单。理想状态就是我把我的myblog项目托管到github，然后每次我写完文章直接push到github仓库，webhook监听到我的push，给我的服务器发送一个http请求，服务器收到请求后执行本地shell脚本，自动拉取最新的仓库代码，然后执行hugo编译成网页。这样就实现自动部署啦！</p>
## 项目代码托管到github

在常用的本地机器上安装git，myblog 目录作为 git 仓库，push 到 github 进行备份。由于 public 目录下的静态网页完全可由其余文件自动生成，因此仓库可以排除 public 目录。

```shell
touch .gitignore && echo "/public" >> .gitignore
git add .
git commit -m "初次提交"
git branch -m main
git push -u origin main
```

## 服务器安装node

使用node可以很简单的启动一个web服务

选择[node版本](https://nodejs.org/en/download/)

node安装依次执行以下命令：

```shell
cd ~
wget https://nodejs.org/dist/v14.17.3/node-v14.17.3-linux-x64.tar.xz
tar -xvf node-v14.17.3-linux-x64.tar.xz
cd node-v14.17.3-linux-x64/bin
ln -s /root/node-v14.17.3-linux-x64/bin/node /usr/bin/
ln -s /root/node-v14.17.3-linux-x64/bin/npm /usr/bin/
```

我是centos系统，直接安装编译好的二进制文件了，之前试过自己编译，要等好久就放弃了

安装完后可以执行以下命令检查：

```shell
node -v
npm -v
```

正确输出版本号即可

npm安装成功后，再安装一下pm2，安装后管理进程十分好用

```shell
npm install pm2@latest -g
ln -s /root/node-v14.17.3-linux-x64/bin/pm2 /usr/bin/
```

执行命令查看是否安装成功

```shell
pm2 -v
```

## 配置脚本

新建一个目录`webhook`，脚本文件放在这里

### 编写shell脚本

```shell
cd ~
mkdir webhook
cd webhook
touch hugo-deploy.sh
vi hugo-deloy.sh
```

按i进入编辑模式，我的脚本如下（我的仓库就是`myblog`，然后hugo生成的静态文件就在`~/myblog/public`下，也是我的网站根目录。如果网站根目录不在这里，hugo也可以指定生成的目录，使用`hugo -d /dir`）

```shell
#!/bin/bash

cd ~/myblog
git pull origin main
hugo 
echo "deploy success!"
exit 0
```

按下Esc键，并输入`:wq`保存退出文件。

给文件添加可执行权限

```bash
chmod +x hugo-deploy.sh
```

### Github配置SSH Key

GitHub配置SSH Key的目的是为了帮助我们在通过git拉取代码时，不需要繁琐的验证用户名密码，如果需要验证，我们的自动脚本就挂了。

首先检查是否存在sshkey

```shell
cd ~/.ssh
ls
# 或者
ll
# 看是否存在 id_rsa 和 id_rsa.pub文件，如果存在，说明已经有SSH Key
```

如果没有则执行如下命令，然后回车到底

```shell
ssh-keygen
```

最后获取sshkey填入github配置中（点击右上角头像，settings，找到ssh点进去取个名字复制下即可。）

```shell
cat ~/.ssh/id_rsa.pub
```

最后检查下服务器仓库的`.git`文件夹config文件是否是ssh提交，如果不是可以改掉

```shell
cat ~/.git/config
```

输出

```
[core]
	repositoryformatversion = 0
	filemode = true
	bare = false
	logallrefupdates = true
[remote "origin"]
	url = https://github.com/K1ngram4/myblog.git
	fetch = +refs/heads/*:refs/remotes/origin/*
```

这里url改为`git@github.com:K1ngram4/myblog.git`这个地址可以到仓库下载代码的地方选择ssh复制。

### 编写js脚本

先安装一个第三方插件[github-webhook-handler](https://github.com/rvagg/github-webhook-handler)

```bash
npm install github-webhook-handler --save
```

然后在webhook目录下创建一个`github-webhook.js`文件，写入以下内容，就是监听github的webhook请求

```js
var http = require('http')
var createHandler = require('github-webhook-handler')
var exec = require('child_process').exec
var handler = createHandler({ path: '/webhook', secret: 'mysecret' })

http.createServer(function (req, res) {
  handler(req, res, function (err) {
    res.statusCode = 404
    res.end('no such location')
  })
}).listen(7777)

console.log("github Hook Server running at http://0.0.0.0:7777/webhook");

handler.on('error', function (err) {
  console.error('Error:', err.message)
})

handler.on('push', function (event) {
  console.log('Received a push event for %s to %s',
    event.payload.repository.name,
    event.payload.ref)
    exec('sh ./hugo-deploy.sh', function (error, stdout, stderr) {
        if(error) {
            console.error('error:\n' + error);
            return;
        }
        console.log('stdout:\n' + stdout);
        console.log('stderr:\n' + stderr);
    });
})
```

记住这个地址`http://kingram.top:7777/webhook`这就是webhook要发送过来的地址，下一步需要用到。

### 启动脚本

```bash
node github-webhook.js
```

这样启动可以用来调试，如果调试没问题后，这样启动

```bash
pm2 start github-webhook.js
pm2 startup
```

## github配置webhook

打开托管仓库`github.com/K1ngram4/myblog`点击右上角`Settings`按钮，选择`Webhooks`，点击右上角`Add wehook`

Payload URL 填服务器端创建好的`http://kingram.top:7777/webhook`

Content type选择`application/json`

Secret填入`mysecret`



<p>到此，所有配置就结束了，可以在本机上push到仓库，服务器就会自动编译网站了。(#^.^#)</p>
博客地址：[传送门](http://kingram.top)