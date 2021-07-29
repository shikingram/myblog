{
  "title":"git的使用（二）git冲突解决和版本回退",
  "description":"《github入门与实践》阅读笔记",
  "tags":[
    "git & github"
  ],
  "date":"2021-07-28",
  "lastmod":"2021-07-28",
  "draft":"false",
  "author":"kingram"
}

## 版本回退

### git reset——回溯历史版本

回到创建`feature-A`分支之前的`master`分支，创建一个名为`fix-B`的分支

首先查看当前分支图表

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

通过这个命令可以看到创建`feature-A`分支前的时间点的hash值为`6ab6d6b65bc4d28be5b571dbc01df81e2231effa`，只要执行`git reset --hard +hash`就可以回到该hash对应的时间点

```sh
$ git reset --hard 6ab6d6b65bc4d28be5b571dbc01df81e2231effa
HEAD is now at 6ab6d6b add index
```

查看下`README.md`，还没有新建分支

```sh
$ cat README.md
# GIT测试
```

创建分支`fix-B`

```sh
$ git switch -c fix-B
Switched to a new branch 'fix-B'
$ git branch
  feature-A
* fix-B
  master
```

修改`README.md`，增加一行

```
# GIT测试
- fix-B
```

提交

```sh
$ git add README.md
$ git commit -m "fix B"
[fix-B 8d19107] fix B
 1 file changed, 1 insertion(+)
```

查看下当前版本库的分支图表

```sh
$ git log --graph
* commit 8d191079fac23089ae61d916956dee27faaa4bc2 (HEAD -> fix-B)
| Author: kingram <kingram@163.com>
| Date:   Wed Jul 28 20:05:25 2021 +0800
|
|     fix B
|
* commit 6ab6d6b65bc4d28be5b571dbc01df81e2231effa (master)
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

现在要做的就是将fix-B合并到master中，如图中红线标记。注意不是在当前状态直接合并，当前状态切换到master再合并fix-B就没有feature-A分支修改的内容了。

此时在历史记录中，版本库在创建`feature-A`前的时间点产生了两个分支，一个是`feature-A`的线，一个是`fix-B`的线，两条线其实是平行空间的，`feature-A`的线已经和`master`合并了，而`fix-B`的线还没有合并。

![](/static/img/git/github_part2/1.jpg)

使用`git reflog`命令查看仓库操作日志，找到合并`feature-A`分支后的时间点的hash值

```sh
$ git reflog
8d19107 (HEAD -> fix-B) HEAD@{0}: commit: fix B
6ab6d6b (master) HEAD@{1}: checkout: moving from master to fix-B
6ab6d6b (master) HEAD@{2}: reset: moving to 6ab6d6b65bc4d28be5b571dbc01df81e2231effa
a4dc01a HEAD@{3}: merge feature-A: Merge made by the 'recursive' strategy.
6ab6d6b (master) HEAD@{4}: checkout: moving from feature-A to master
64aa165 (feature-A) HEAD@{5}: checkout: moving from master to feature-A
6ab6d6b (master) HEAD@{6}: checkout: moving from feature-A to master
64aa165 (feature-A) HEAD@{7}: commit: add feature-A
6ab6d6b (master) HEAD@{8}: checkout: moving from master to feature-A
6ab6d6b (master) HEAD@{9}: commit: add index
7419956 HEAD@{10}: commit (initial): first commit
```

可以看到`merge feature-A`的时间点的hash值为`a4dc01a`

先切换到`master`分支，再回退到`a4dc01a`时间点 

```
$ git switch master
Switched to branch 'master'
$ git reset --hard a4dc01a
HEAD is now at a4dc01a Merge branch 'feature-A'
```

此时状态如图所示：

![](/static/img/git/github_part2/2.jpg)

现在只要合并`fix-B`分支到`master`分支，就完成任务了，但是我们在`feature-A`分支中也是修改了`README.md`文件的，所以合并会产生冲突。

```sh
$ git merge fix-B
Auto-merging README.md
CONFLICT (content): Merge conflict in README.md
Automatic merge failed; fix conflicts and then commit the result.
```

`CONFLICT`中显示`README.md`文件产生了冲突，查看下文件

```sh
$ cat README.md
# GIT测试

<<<<<<< HEAD
- feature-A

=======
- fix-B
>>>>>>> fix-B
```

`=======`上面的是当前HEAD内容，下面是要合并的内容。修改如下

```sh
$ cat README.md
# GIT测试
- feature-A
- fix-B
```

解决冲突后提交一下文件

```sh
$ git add README.md
$ git commit -m "fixed conflict"
[master d521ac7] fixed conflict
```

#### git commit --amend——修改提交信息

上一条提交信息标记了为`fix conflict`，但其实是fix-B分支的合并，即：`Merge branch 'fix-B'`

查看`git log`

```sh
commit d521ac7386bc3aebd58d41c6420d29196751127a (HEAD -> master)
Merge: a4dc01a 8d19107
Author: kingram <kingram@163.com>
Date:   Thu Jul 29 09:24:12 2021 +0800

    fixed conflict

commit 8d191079fac23089ae61d916956dee27faaa4bc2 (fix-B)
Author: kingram <kingram@163.com>
Date:   Wed Jul 28 20:05:25 2021 +0800

    fix B

commit a4dc01ab44706349cc7661e7b1c4315d1b6fd405
Merge: 6ab6d6b 64aa165
Author: kingram <kingram@163.com>
Date:   Wed Jul 28 18:55:42 2021 +0800

    Merge branch 'feature-A'

commit 64aa16533ca53ae652aca4f6ce8227b7aabc4ad4 (feature-A)
Author: kingram <kingram@163.com>
Date:   Wed Jul 28 18:44:46 2021 +0800

    add feature-A

commit 6ab6d6b65bc4d28be5b571dbc01df81e2231effa
Author: kingram <kingram@163.com>
Date:   Wed Jul 28 18:23:10 2021 +0800

    add index

commit 7419956521d4a00e4fa35ec754946f6cdfce10cf
Author: kingram <kingram@163.com>
Date:   Wed Jul 28 17:40:54 2021 +0800

    first commit
```

修改提交信息

```
git commit --amend
```

编译器显示

```sh
fixed conflict

# Please enter the commit message for your changes. Lines starting
# with '#' will be ignored, and an empty message aborts the commit.
#
# Date:      Thu Jul 29 09:24:12 2021 +0800
#
# On branch master
# Changes to be committed:
#       modified:   README.md
#
```

按下`i`，修改为`Merge branch 'fix-B'`，按下`:wq`保存退出

```sh
[master a564645] Merge branch 'fix-B'
 Date: Thu Jul 29 09:24:12 2021 +0800
```

查看当前分支图表

```sh
$ git log --graph
*   commit a564645c1cc0bb1f57512fe077abaeaa269aad84 (HEAD -> master)
|\  Merge: a4dc01a 8d19107
| | Author: kingram <kingram@163.com>
| | Date:   Thu Jul 29 09:24:12 2021 +0800
| |
| |     Merge branch 'fix-B'
| |
| * commit 8d191079fac23089ae61d916956dee27faaa4bc2 (fix-B)
| | Author: kingram <kingram@163.com>
| | Date:   Wed Jul 28 20:05:25 2021 +0800
| |
| |     fix B
| |
* |   commit a4dc01ab44706349cc7661e7b1c4315d1b6fd405
|\ \  Merge: 6ab6d6b 64aa165
| |/  Author: kingram <kingram@163.com>
|/|   Date:   Wed Jul 28 18:55:42 2021 +0800
| |
| |       Merge branch 'feature-A'
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

#### git rebase -i——压缩历史

在合并特性分支之前，如果发现已提交的内容中有些许拼写错误等，不妨提交一个修改，然后将这个修改包含到前一个提交之中，压缩成一个历史记录。这是个会经常用到的技巧。

```sh
$ git switch -c feature-C
Switched to a new branch 'feature-C'
```

修改`README.md`文件

```
# GIT测试
- feature-A
- fix-B
- faeture-C
```

这里留了个小拼写错误

提交

```sh
$ git commit -am "add feature-C"
[feature-C 768f68a] add feature-C
 1 file changed, 1 insertion(+), 1 deletion(-)
```

修改`README.md`文件拼写错误

修改后查看区别

```sh
$ git diff HEAD
diff --git a/README.md b/README.md
index f583f41..4f997d5 100644
--- a/README.md
+++ b/README.md
@@ -1,4 +1,4 @@
 # GIT测试
 - feature-A
 - fix-B
-- faeture-C
+- feature-C
```

提交

```sh
$ git commit -am "fix typo"
[feature-C 65dd8a2] fix typo
 1 file changed, 1 insertion(+), 1 deletion(-
```

修改历史

```sh
git rebase -i HEAD~2
```

修改编译器内容

```sh
pick 768f68a add feature-C
pick 65dd8a2 fix typo

# Rebase a564645..65dd8a2 onto a564645 (2 commands)
#
# Commands:
# p, pick <commit> = use commit
# r, reword <commit> = use commit, but edit the commit message
# e, edit <commit> = use commit, but stop for amending
# s, squash <commit> = use commit, but meld into previous commit
# f, fixup <commit> = like "squash", but discard this commit's log message
# x, exec <command> = run command (the rest of the line) using shell
# b, break = stop here (continue rebase later with 'git rebase --continue')
# d, drop <commit> = remove commit
# l, label <label> = label current HEAD with a name
# t, reset <label> = reset HEAD to a label
# m, merge [-C <commit> | -c <commit>] <label> [# <oneline>]
# .       create a merge commit using the original merge commit's
# .       message (or the oneline, if no original merge commit was
# .       specified). Use -c <commit> to reword the commit message.
#
# These lines can be re-ordered; they are executed from top to bottom.
#
# If you remove a line here THAT COMMIT WILL BE LOST.
#
# However, if you remove everything, the rebase will be aborted.
```

按下`i`将`fixtypo`的`pick`改为`fixup`，按下`:wq`保存退出

```sh
Successfully rebased and updated refs/heads/feature-C.
```

系统显示rebase成功，将"Fix typo"的内容合并到了上一个提交 "Add feature-C"中，改写成了一个新的提交。这样一来，Fix typo就从历史中被抹去，也就相当于Add feature-C中从来没有出现过拼写错误。这算是一种良性的历史改写。

合并分支

```sh
$ git merge feature-C
Updating a564645..cc1fcc4
Fast-forward
 README.md | 2 +-
 1 file changed, 1 insertion(+), 1 deletion(-)
```



