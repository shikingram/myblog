---
title: "设置两个git配置，解决 git@github.com: Permission denied"   
author: "Kingram"  
date: 2022-04-06   
lastmod: 2022-04-06

tags: [  
"git",
]
---

## 报错信息如下：
```shell
git@github.com: Permission denied (publickey).
Could not read from remote repository.

Please make sure you have the correct access rights
and the repository exists.
```

**如果你试过了重新生成rsa还不能解决问题**，请直接查看``` ~/.ssh/config``中的Host配置，网上很多教程都说这个Host可以随便配置，但是你试试把Host别名改为托管仓库的域名或者ip，即github(假设你的别名是这个)改为github.com，你会发现问题解决了。

这个文件的git配置方式整理如下：

一般情况下，我们的工作机器都会设置两个git的配置，一个是Gitlab的用于工作，一个是github的用于摸鱼，配置方式如下：

### 设置gitlab秘钥：
1. $ssh-keygen -t rsa -C "你的gitlab邮箱"
2. 输入 id_rsa_gitlab取名，最后会生成id_rsa_gitlab 与id_rsa_gitlab.pub两个文件
3. 回车
4. cd ~/.ssh
5. vim 与id_rsa_gitlab.pub 拷贝到gitlab设置ssh keys

### 设置github秘钥：
1. $ssh-keygen -t rsa -C "kingram@163.com"
2. 输入id_rsa_github，会生成id_rsa_github 与id_rsa_github.pub两个文件
3. 回车
4. cd ~/.ssh
5. vim id_rsa_github.pub 拷贝到github设置ssh keys

如果出现 could not open a connection to your authentication agent的错误，执行如下：
```shell
ssh-agent bash
ssh-add id_rsa_xxx  (xxx替换具体的名字)
```

### 配置config：
1. cd ~/.ssh
2. touch config （新建config文件，没有后缀）
3. 设置config内容：

```shell
Host git.xxx.com
HostName git.xxx.com
IdentityFile ~/.ssh/id_rsa_gitlab


Host github.com
HostName github.com
IdentityFile ~/.ssh/id_rsa_github
```
- Host: 别名
- HostName: 托管仓库的域名或者ip
- IdendityFile: 秘钥的路径

### 测试配置是否成功：
1. ssh -T git@git.xxx.com
2. ssh -T git@github.com
注：此处的 git.xxx.com 、 github.com 是config内容的Host别名，**_别名请设置和HostName一致_**。

### 用户名和邮箱
- git config -l
如果开始设置过如：
```shell
git config --global user.name "xxx"
git config --global user.email "xxx@yy.com"
```
需要去掉
- 去掉全局设置：
```shell
git config --global --unset user.name
git config --global --unset user.email
```
- 设置用户和邮箱
进入到项目的根目录：cd ~/workspace/project
```shell
git config user.name "kingram"
git config user.email "kingram@163.com"
```
注： 这一步用户名和邮箱不设置的话，一般编译器一般都会提示设置，比如goland就会自动提示。
