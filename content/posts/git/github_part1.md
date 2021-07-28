{   

 "title": "github的使用（一）",   

 "description": "《github入门与实践》阅读笔记",  

 "tags": [ "git & github"],   

"date": "2021-07-26",  

"lastmod": "2021-07-26",

"draft":"true",

 }

## 什么是版本管理

版本管理就是管理更新的历史记录。

- 集中型与分散型

集中型将所有数据集中存放在服务器当中，有便于管理的优点。但是一旦开发者所处的环境不能连接服务器，就无法获取最新的源代码，开发也就几乎无法进行。万一服务器故障导致数据消失，恐怕开发者就再也见不到最新的源代码了

分散型拥有多个仓库，相对而言稍显复杂。不过，由于本地的开发环境中就有仓库，所以开发者不必连接远程仓库就可以进行开发。

## 初始设置

- 设置姓名和邮箱地址

```shell
git config --global user.name "yourname"
git config --global user.email "youremail@163.com"
```

这个命令，会在“～/.gitconfig”中以如下形式输出设置文件。

```
[user]
	name = yourname
	email = youremail@163.com
```

想更改这些信息时，可以直接编辑这个设置文件。

这里的邮箱名字是公开的而且是有效的在github上验证过的，如果这个邮箱和你的github账号绑定的邮箱不一致，那你的提交记录就不会在小方格中显示为绿色。也可以在setting->email中添加一个邮箱并验证，只要邮箱有效，那么小方格就会变绿了。

提高命令输出的可读性

将color.ui设置为auto可以让命令的输出拥有更高的可读性。

```shell
git config --global color.ui auto
```

“～/.gitconfig”中会增加一行

```
[color]
	ui = auto
```

