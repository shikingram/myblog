{
  "title":"git的使用（三）远程仓库操作",
  "description":"《github入门与实践》阅读笔记",
  "tags":[
    "git & github"
  ],
  "date":"2021-08-05",
  "lastmod":"2021-08-05",
  "draft":"false",
  "author":"kingram"
}

## 推送至远程仓库
先在github上建一个仓库，取名和本地仓库一致。
```
git-tutorial
```

创建时不要勾选Add a README file，因为本地有一个，如果创建了则远程和本地不一致。

- git remote add --添加远程仓库

```shell
git remote add origin git@github.com:K1ngram4/git-tutorial.git
```

- git push --推送至远程仓库

**推送至master分支**

```shell
$ git push -u origin master
Enumerating objects: 19, done.
Counting objects: 100% (19/19), done.
Delta compression using up to 8 threads
Compressing objects: 100% (7/7), done.
Writing objects: 100% (19/19), 1.43 KiB | 146.00 KiB/s, done.
Total 19 (delta 1), reused 0 (delta 0), pack-reused 0
remote: Resolving deltas: 100% (1/1), done.
To github.com:K1ngram4/git-tutorial.git
 * [new branch]      master -> master
Branch 'master' set up to track remote branch 'master' from 'origin'.
```

执行改命令后当前分支的内容就会被推送至远程仓库origin的master分支，-u参数可以在推送时，将origin仓库的master分支设置为本地仓库的upstream（上游）。加了这个参数后，git pull 命令就可以直接从origin master分支获取内容。

**推送至master以外的分支**

在本地仓库创建feature-D分支

```shell
$ git switch -c feature-D
Switched to a new branch 'feature-D'
$ git branch
  feature-A
  feature-C
* feature-D
  fix-B
  master
```

推送至远程仓库，并保持分支名称不变

```shell
$ git push -u origin feature-D
Total 0 (delta 0), reused 0 (delta 0), pack-reused 0
remote:
remote: Create a pull request for 'feature-D' on GitHub by visiting:
remote:      https://github.com/K1ngram4/git-tutorial/pull/new/feature-D
remote:
To github.com:K1ngram4/git-tutorial.git
 * [new branch]      feature-D -> feature-D
Branch 'feature-D' set up to track remote branch 'feature-D' from 'origin'.
```

在github页面就可看到两个分支了

## 从远程仓库获取代码

如果我是另一个开发者，我需要获取git-tutorial仓库

- git clone --获取远程仓库

换个目录克隆项目到本地

```shell
$ git clone git@github.com:K1ngram4/git-tutorial.git
Cloning into 'git-tutorial'...
remote: Enumerating objects: 19, done.
remote: Counting objects: 100% (19/19), done.
remote: Compressing objects: 100% (6/6), done.
remote: Total 19 (delta 1), reused 19 (delta 1), pack-reused 0
Receiving objects: 100% (19/19), done.
Resolving deltas: 100% (1/1), done.
```

执行git clone命令后我们会默认处于master分支下，同时，origin就是远程仓库。

进入git-tutorial目录

```shell
$ git branch -a
* master
  remotes/origin/HEAD -> origin/master
  remotes/origin/feature-D
  remotes/origin/master
```

-a 查看本地仓库和远程仓库的分支信息

**获取远程feature-D分支**

```shell
$ git switch -c feature-D origin/feature-D
Switched to a new branch 'feature-D'
Branch 'feature-D' set up to track remote branch 'feature-D' from 'origin'.
```

以origin的仓库的feature-D分支为来源，在本地仓库中创建feature-D分支

**向本地的feature-D分支提交修改**

我现在是另一个开发者，我在feature-D分支上做了修改--在README.md文件中添加一行

```
# GIT测试
- feature-A
- fix-B
- feature-C
- feature-D
```

```shell
$ git diff
diff --git a/README.md b/README.md
index 4f997d5..72a02e5 100644
--- a/README.md
+++ b/README.md
@@ -2,3 +2,4 @@
 - feature-A
 - fix-B
 - feature-C
+- feature-D

```

提交

```shell
$ git commit -am "add feature-D"
[feature-D 013a47c] add feature-D
 1 file changed, 1 insertion(+)

```

**推送至远程feature-D分支**

```shell
$ git push
Enumerating objects: 5, done.
Counting objects: 100% (5/5), done.
Delta compression using up to 8 threads
Compressing objects: 100% (2/2), done.
Writing objects: 100% (3/3), 271 bytes | 271.00 KiB/s, done.
Total 3 (delta 0), reused 0 (delta 0), pack-reused 0
To github.com:K1ngram4/git-tutorial.git
   cc1fcc4..013a47c  feature-D -> feature-D
```

从远程仓库获取feature-D分支，在本地仓库中提交更改，再将feature-D分支推送回远程仓库，通过这一系列操作，就可以与其他开发者相互合作，共同培育feature-D分支，实现某些功能。

- git pull --获取最新的远程仓库分支

现在有个开发者在远程feature-D分支推送了提交，这时我们就可以用git pull命令将本地的feature-D分支更新到最新状态。

回到原来的目录，查看当前分支

```shell
$ git branch
  feature-A
  feature-C
* feature-D
  fix-B
  master
  
```

这样就在feature-D分支下更新就好

```shell
$ git pull origin feature-D
remote: Enumerating objects: 5, done.
remote: Counting objects: 100% (5/5), done.
remote: Compressing objects: 100% (2/2), done.
remote: Total 3 (delta 0), reused 3 (delta 0), pack-reused 0
Unpacking objects: 100% (3/3), 251 bytes | 17.00 KiB/s, done.
From github.com:K1ngram4/git-tutorial
 * branch            feature-D  -> FETCH_HEAD
   cc1fcc4..013a47c  feature-D  -> origin/feature-D
Updating cc1fcc4..013a47c
Fast-forward
 README.md | 1 +
 1 file changed, 1 insertion(+)
```

这样我就把其他开发者提交的代码更新到本地了

但是如果我们修改了同一部分的代码，push时就容易发生冲突，所以多名开发者在同一个分支中作业时，为减少冲突情况发生，建议频繁的push和pull操作。
