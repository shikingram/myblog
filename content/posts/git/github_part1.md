{   
 "title": "git的使用（一）git基本使用和分支操作",   
 "description": "《github入门与实践》阅读笔记",  
 "tags": [ "git & github"],   
 "date": "2021-07-26",  
 "lastmod": "2021-07-28",
 "draft":"false",
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

- 提高命令输出的可读性

将color.ui设置为auto可以让命令的输出拥有更高的可读性。

```shell
git config --global color.ui auto
```

“～/.gitconfig”中会增加一行

```
[color]
	ui = auto
```

## 设置SSH Key

- 创建SSH Key

GitHub上连接已有仓库时的认证，是通过使用了SSH的公开密钥认证方式进行的。运行下面的命令创建SSH Key。

```shell
ssh-keygen -t rsa -C "youremail@163.com"
```

- 获取公有密钥

```shell
cat ~/.ssh/id_rsa.pub
```

打开github设置，点击SSHKeys菜单，添加SSH Key，key中粘贴公有秘钥

添加成功之后，创建账户时所用的邮箱会接到一封提示“公共密钥添加完成”的邮件。

- 测试认证成功

```shell
ssh -T git@github.com
```

连接成功后，接下来每次连接github通过密钥进行认证，不需要再输入用户名和密码，简单方便。

## Git使用

### Git基本操作

#### git init——初始化仓库

要使用Git进行版本管理，必须先初始化仓库。

新建`git-tutorial`目录后，执行`git init`命令，就将该目录使用git进行版本管理了。

```shell
$ git init
Initialized empty Git repository in C:/Users/shiku/Desktop/myproject/git-tutorial/.git/
```

如果初始化成功，执行了`git init`命令的目录下就会生成．git目录。在Git中，我们将这个目录的内容称为“附属于该仓库的**工作树**”。文件的编辑等操作在工作树中进行，然后记录到仓库中，以此管理文件的历史快照。如果想将文件恢复到原先的状态，可以从仓库中调取之前的快照，在工作树中打开。

#### git status——查看仓库的状态

```shell
$ git status
On branch master

No commits yet

nothing to commit (create/copy files and use "git add" to track)
```

工作树和仓库在被操作的过程中，状态会不断发生变化。在Git操作过程中时常用git status命令查看当前状态。

新建一个文件`README.md`，再执行`git status`

```sh
$ git status
On branch master

No commits yet

Untracked files:
  (use "git add <file>..." to include in what will be committed)
        README.md

nothing added to commit but untracked files present (use "git add" to track)

```

#### git add——向暂存区中添加文件

新建文件会显示在`Untracked files`中，该文件不会被记录在git仓库的版本管理对象中，需要使用git add命令将其加入**暂存区**中。加入后就显示在`Changes to be committed`中了。

```sh
$ git add README.md
$ git status
On branch master

No commits yet

Changes to be committed:
  (use "git rm --cached <file>..." to unstage)
        new file:   README.md

```

#### git commit——保存仓库的历史记录

git commit命令可以将当前暂存区中的文件实际保存到仓库的历史记录中。通过这些记录，我们就可以在工作树中复原文件。

```sh
$ git commit -m "first commit"
[master (root-commit) 7419956] first commit
 1 file changed, 0 insertions(+), 0 deletions(-)
 create mode 100644 README.md
```

-m参数是提交信息

#### git log——查看提交日志

```sh
$ git log
commit 7419956521d4a00e4fa35ec754946f6cdfce10cf (HEAD -> master)
Author: kingram <kingram@163.com>
Date:   Wed Jul 28 17:40:54 2021 +0800

    first commit
```

- git log --pretty=short    #只显示提交信息的第一行

```sh
$ git log --pretty=short
commit 7419956521d4a00e4fa35ec754946f6cdfce10cf (HEAD -> master)
Author: kingram <kingram@163.com>

    first commit
```

- git log +目录/文件名   #显示指定目录、文件的日志

```sh
$ git log README.md
commit 7419956521d4a00e4fa35ec754946f6cdfce10cf (HEAD -> master)
Author: kingram <kingram@163.com>
Date:   Wed Jul 28 17:40:54 2021 +0800

    first commit
```

- git log -p +文件名  #显示文件的改动

```sh
$ git log -p README.md
commit 7419956521d4a00e4fa35ec754946f6cdfce10cf (HEAD -> master)
Author: kingram <kingram@163.com>
Date:   Wed Jul 28 17:40:54 2021 +0800

    first commit

diff --git a/README.md b/README.md
new file mode 100644
index 0000000..e69de29
```

#### git diff——查看更改前后的差别

在`README.md`中写入

```
# GIT测试
```

- git diff #查看**工作树**和**暂存区**的差别

```sh
$ git diff
diff --git a/README.md b/README.md
index e69de29..9919bd3 100644
--- a/README.md
+++ b/README.md
@@ -0,0 +1,2 @@
+# GIT测试
+
```

*当前状态：暂存区是空的，工作树是写入了GIT测试的README.md，所以程序只会显示工作树和最新提交状态之间的差别*

- git diff HEAD #查看工作树和最新提交的差别

```sh
$ git diff HEAD
diff --git a/README.md b/README.md
index e69de29..9919bd3 100644
--- a/README.md
+++ b/README.md
@@ -0,0 +1,2 @@
+# GIT测试
+
```

此时把README.md的修改添加到暂存区中

```shell
git add README.md
```

*当前状态：暂存区和工作树一致，所以执行`git diff`不会有输出，但执行`git diff HEAD`会展示工作树和最新提交状态之间的差别*

```shell
$ git diff HEAD
diff --git a/README.md b/README.md
index e69de29..9919bd3 100644
--- a/README.md
+++ b/README.md
@@ -0,0 +1,2 @@
+# GIT测试
+
```

不妨养成这样一个好习惯：在执行git commit命令之前先执行git diff HEAD命令，查看本次提交与上次提交之间有什么差别，等确认完毕后再进行提交。

这里的HEAD是指向当前分支中最新一次提交的指针。

确认差别后，直接运行git commit命令提交。

```sh
$ git commit -m "add index"
[master 6ab6d6b] add index
 1 file changed, 2 insertions(+)
```

### 分支操作

#### git branch——显示分支一览表

```sh
$ git branch
* master
```

可以看到master分支左侧标有“*”（星号），表示这是我们当前所在的分支。

#### git switch -c——创建、切换分支

如果想以当前的master分支为基础创建新的分支，我们需要用到`git switch -c`命令。

```sh
git switch -c feature-A

```

```sh
$ git switch -c feature-A
Switched to a new branch 'feature-A'
```

等同于以下两个命令

```sh
git branch feature-A #创建feature-A分支
git switch feature-A #切换feature-A分支
```

查看下当前分支列表，目前处于feature-A分支下

```sh
$ git branch
* feature-A
  master
```

修改README.md，添加一行

```
# GIT测试
- feature-A
```

提交代码

```sh
$ git add README.md
$ git commit -m "add feature-A"
[feature-A 64aa165] add feature-A
 1 file changed, 2 insertions(+)
```

切换到master分支，查看README.md

```sh
$ git switch master
Switched to branch 'master'
$ git branch
  feature-A
* master
$ cat README.md
# GIT测试
```

可以看到，master分支没有受到影响

切换回上一个分支

```sh
$ git switch -
Switched to branch 'feature-A'
```

**特性分支**

特性分支顾名思义，是集中实现单一特性（主题），除此之外不进行任何作业的分支。

在日常开发中，往往会**创建数个特性分支**，同时在此之外再保留一个随时可以发布软件的稳定分支。稳定分支的角色通常由master分支担当

基于特定主题的作业在特性分支中进行，主题完成后再与master分支合并。只要保持这样一个开发流程，就能保证master分支可以随时供人查看。这样一来，其他开发者也可以放心大胆地从master分支创建新的特性分支。

#### git merge——合并分支

git merge命令用于合并指定分支到当前分支。比如我在feature-A实现完成了，想要将它合并到主干分支master中。首先切换到master分支。

```sh
$ git switch master
Switched to branch 'master'
$ git branch
  feature-A
* master
```

然后合并feature-A分支。为了在历史记录中明确记录下本次分支合并，我们需要创建合并提交。因此，在合并时加上--no-ff参数。

```shell
$ git merge --no-ff feature-A
```

编译器内容不需要修改

按下`:wq`保存退出

```sh
Merge made by the 'recursive' strategy.
 README.md | 2 ++
 1 file changed, 2 insertions(+)
```

#### git log --graph——以图表形式查看分支

```sh
$ git log --graph
*   commit a4dc01ab44706349cc7661e7b1c4315d1b6fd405 (HEAD -> master)
|\  Merge: 6ab6d6b 64aa165
| | Author: kingram <kingram@163.com>
| | Date:   Wed Jul 28 18:55:42 2021 +0800
| |
| |     Merge branch 'feature-A'
| |
| * commit 64aa16533ca53ae652aca4f6ce8227b7aabc4ad4 (feature-A)
|/  Author: kingram <kingram@163.com>
|   Date:   Wed Jul 28 18:44:46 2021 +0800
|
|       add feature-A
|
* commit 6ab6d6b65bc4d28be5b571dbc01df81e2231effa
| Author: kingram <kingram@163.com>
| Date:   Wed Jul 28 18:23:10 2021 +0800
|
|     add index
|
* commit 7419956521d4a00e4fa35ec754946f6cdfce10cf
  Author: kingram <kingram@163.com>
  Date:   Wed Jul 28 17:40:54 2021 +0800

      first commit
```





