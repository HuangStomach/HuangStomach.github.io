---
title: Git提交与修改
date: 2019-08-05 10:27:14
tags:
- Git
- Shell
categories:
- Git
---

> Git要求开发者在编写提交的时候，尽量要填写有用的信息；在工作中更甚，提交写的完善完备更能让问题追踪变得容易可控，不过可能会遇到不小心写错了提交内容的情况，这时候就一定要在推送之前想办法把错误的提交信息修正掉。

## 提交

Git提交是一件非常简单的事情，对于我来说不过是先检查一下需要提交的文件，然后直接一个`git commmit`就结束了的工作，当敲下回车，就会出现熟悉的界面：

``` vim
# Please enter the commit message for your changes. Lines starting
# with '#' will be ignored, and an empty message aborts the commit.
#
# Date:      Mon Aug 5 10:17:05 2019 +0800
#
# On branch master
# Your branch is ahead of 'origin/master' by 1 commit.
#   (use "git push" to publish your local commits)
#
# Changes to be committed:
#       new file:   source/_posts/Git提交与修改.md
#
```

其中第一行就是提交描述信息，也是最重要的地方；第六行是提交时间，第十二行开始则是本次提交修改的文件概览。

有的时候开发者比较自信或者非常熟练，直接使用`git commit -m <message>`来进行快速提交。其实commit中除了`-m --message`之外还可以快速设置一些其他的信息:

``` bash
Commit message options
    --author <author>             override author for commit
    --date <date>                 override date for commit
    -m, --message <message>       commit message
    -c, --reedit-message <commit> reuse and edit message from specified commit
    -C, --reuse-message <commit>  reuse message from specified commit
    --reset-author                the commit is authored by me now (used with -C/-c/--amend)
```

这边随便列举了几个，如我们可以快速设置作者、时间，甚至使用`--reset-author`来把某个提交的作者设置成自己，这个参数的说明很有意思，而且后面标注了，要伴随`-C/-c/--amend`使用，这是什么参数？

## 修改提交

我们使用`man`命令来查看一下说明，可以看到其实和：

``` bash
$ git reset --soft HEAD^
$ #... do something else to come up with the right tree ...
$ git commit -c ORIG_HEAD
```

是等价的，实际上是创建了一个新的提交而不是直接修改上一个提交，当我键入`git commit --amend`之后，就会出现熟悉的界面了，这时候就可以把错误的提交信息修正掉了。

同时也可以和其他快捷参数一起使用，比如说你不想看你之前提交信息写的究竟是什么，也可以直接使用

``` bash
$ git commit --amend -m <message>
```

来快速修改提交信息，同时也可以根据上述参数描述进行修改，比如修改时间：

``` bash
$ git commit --amend --date <date>
```

不过这里需要注意的是这里的参数是特定的时间格式，可以尝试使用`date`，不过不能直接使用`date`命令，因为此处的参数是要求符合`RFC 2822`标准的，所以需要使用如下命令来查看：

``` bash
$ date -R
Mon, 05 Aug 2019 10:50:35 +0800
```

然后再根据这个格式修改时间就可以完成修改commit提交时间的操作了

``` bash
$ git commit --amend --date=`Mon, 05 Aug 2019 10:50:35 +0800`
```
