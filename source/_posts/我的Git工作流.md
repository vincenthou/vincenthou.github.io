git现在使用的人越来越多，和传统的cvs，svn等工具相比有很多的[优势和特性](http://www.oschina.net/news/12542/git-and-svn)。当
你开始使用起来之后，你会发现自己逐渐的爱上她。基本的git操作这里就不赘述了，
查阅[参考文档](http://gitref.org/)，或者一些[简明教程](http://rogerdudler.github.io/git-guide/index.zh.html)，就能大致的了解，最关键是自己建一个自己的
库，多加练习。

[GitHub](https://github.com/)是全球最大的社交编程及代码托管网站，GitHub可以托管各种git库，并提
供一个web界面，但与其它像[SourceForge](http://sourceforge.net/)或[Google Code](http://code.google.com/intl/zh-CN/)这样的服务不同，GitHub的
独特卖点在于从另外一个项目进行分支的简易性而且主要是使用git作为管理工具。

在github上可以找到很多优秀的开源软件，这是一个学习的好机会，我们也可以坚
持拿来主义，大量的优秀软件帮助我们解决开发中遇到的难题。同时这也是一个共享
与交流的平台，你不仅仅可以学习和使用别人的代码，还能创建自己的库，向别人的
库中提交代码，为开源社区贡献自己的力量。这里就说一下一般是如何向已有的库提
交代码的。
#1. Fork 你想要贡献代码的库

![image](https://github.s3.amazonaws.com/docs/bootcamp_3_fork.jpg)
找到你想要贡献代码的库的主页，点击图中标出来的fork按钮，这样就会在你自己的库
中添加一个和你fork的库（我们叫它upstream）同名的库（我们叫它mystream）。
#2. 克隆你fork的库

使用git clone命令将mystream克隆到本地。

``` bash
git clone git@github.com:<username>/<repo-name>.git
```

这个克隆出来的库只是你github上面的库在本地一个副本，你可以对它任意的操作，
只要不push你的改动，是不会对你github上面的代码产生任何影响，同时也体现了git
离线提交的好处。
#3. 在remote列表中添加你要贡献代码的库

既然要贡献你的代码给别人，就要先把别人的地址给记下来吧，使用git remote add命
令来添加upstream。

``` bash
git remote add upstream <original repo git url>
```

到这里主要的准备工作就做完了。可以使用git remote命令查看你的remote列表。
#4. 创建一个工作分支，在上面完成你的工作

使用git和别人协同工作一个很重要的原则是不要在主分支上进行修改，这个原则和后
面提到的原子性提交以及pull request 都是有关系的，后面会说到。首先确认你在主分
支上

``` bash
git checkout master
```

创建并且切换到一个新的工作分支

``` bash
git checkout -b <new-branch-name>
```

在这个工作分支上你就可以为所欲为了，你可以添加，修改代码，并且进行正常的
commit提交
#5. 保持你的代码同步

因为我们的开发和修改是独立的，和我们fork的代码库是并行进行的，所以在我们fork
后，或者上次同步一段时间后，我们本地的代码已经不是最新的了。当你完成了修改
并且想向主库提交代码时，首先要做的事情就是同步你的代码。
确认你工作在主分支上

``` bash
git checkout master
```

从upstream的主代码库获取最新的代码并且与本地的主分支合并

``` bash
git pull upstream master
```

这是懒人的做法，你也可以选择使用git fetch和git merge，选择性合并主分支的代
码，这里有它们的[区别](http://www.tech126.com/git-fetch-pull/)另外，你也可以将你
自己在github上的mystream主分支更新，当然这个是可选的。

``` bash
git push origin master
```

一般默认你克隆下来本地对应的远端mystream被命名为origin
#6. 同步你想要提交的分支代码

到这里你本地的主分支已经和upstream的主分支同步了，真正的好戏才刚刚开始。前
面提到提交的原子性，那什么是提交的原子性呢？我们都知道乐高积木，这个提交的
原子性就是保证我们使用乐高积木拼装成的艺术品，随时可以进行调整。每一次提交
就像一个积木块一样，可以随时拔掉或者插上。

但是当我们在本地为了添加一个功能或者修复一个bug，很多时候不会只做一次提交，
而是可能会进行多次commit提交。如果直接将这些改动与upstream主分支合并，那么
以后如果希望撤销这次的操作或者将这次的改动合并到其他分支上就会很麻烦。

首先确保你在你一直辛勤工作的分支

``` bash
git checkout <working-branch-name>
```

将你本地的工作分支与master主分支“合并”

``` bash
git rebase -i master
```

这里用到的rebase命令不是简单的合并，这里有[rebase与merge的区别](http://www.liuhui998.com/4_2.html)，-i参数会引导
我们一步一步解决冲突，你也可以选择先解决冲突再使用rebase。使用rebase命令会
将你原来多次的提交合并成一次提交。
其中-i 参数会用交互的方式引导你合并为一次提交，一般你会看到这样的编辑界面

``` bash
pick ff76694 add line 3
pick 8e49657 add line 4
# Rebase d879706..8e49657 onto d879706
#
# Commands:
#  p, pick = use commit
#  r, reword = use commit, but edit the commit message
#  e, edit = use commit, but stop for amending
#  s, squash = use commit, but meld into previous commit
#  f, fixup = like "squash", but discard this commit's log message
#  x, exec = run command (the rest of the line) using shell
#
# These lines can be re-ordered; they are executed from top to bottom.
#
# If you remove a line here THAT COMMIT WILL BE LOST.
#
# However, if you remove everything, the rebase will be aborted.
#
# Note that empty commits are commented out
```

下面的注释中Command有详细的解释，一般将你要合并的多个提交前面的pick改为squash（如果有多个commit除了第一个是pick其他都是squash），对于我们的示例，修改为

``` bash
pick ff76694 add line 3
squash 8e49657 add line 4
# Rebase d879706..8e49657 onto d879706
#...
```

保存文件退出编辑，正常情况下这样就完事了，但是如果master分支上有新的提交和你的工作分支提交发生冲突，就会看到下面的情况

``` bash
error: could not apply ff76694... add line 3

When you have resolved this problem, run "git rebase --continue".
If you prefer to skip this patch, run "git rebase --skip" instead.
To check out the original branch and stop rebasing, run "git rebase --abort".
Could not apply ff76694624019140e05cd9d443aa547e62c5c24b... add line 3
```

编辑冲突文件（冲突文件中可能不会列出你在当前分支上所有的改动，只会标出冲突部分），选择需要的部分，保存文件

``` bash
git add <modified files>
git rebase --continue
```

之后你会看到类似这样的提示

``` bash
add line 3

# Please enter the commit message for your changes. Lines starting
# with '#' will be ignored, and an empty message aborts the commit.
# rebase in progress; onto d879706
# You are currently rebasing branch 'feature' on 'd879706'.
#
# Changes to be committed:
#       modified:   test.txt
#
```

这里是让你编辑你第一次提交的comment，更改comment内容后保存退出，记着会提示你编辑合并commit的comment，你在上一次添加的comment会合并进来，你看到的会是这样的

``` bash
# This is a combination of 2 commits.
# The first commit's message is:

add line 3

# This is the 2nd commit message:

add line 4

# Please enter the commit message for your changes. Lines starting
# with '#' will be ignored, and an empty message aborts the commit.
# rebase in progress; onto d879706
# You are currently editing a commit while rebasing branch 'feature' on 'd879706'.
#
# Changes to be committed:
#       modified:   test.txt
#
```

去掉多余的comment内容，保存退出。
到这里工作分支和master的合并就完成了，使用rebase，你只在工作分支上做到了两件事：
1. 同步本地master的最新代码到工作分支
2 .将工作分支上的多次提交合并成一次提交，同时append到时间轴的最后
更详细的说明看[这里](http://gitready.com/advanced/2009/02/10/squashing-commits-with-rebase.html)
**这时你在新的分支上的改动还没有同步到本地master**，你需要切换回master分支，将改动合并回来

``` bash
git checkout master
git merge <working-branch-name>
```

你看到maser上的时间轴将会是优雅的直线
#7. 提交本地分支代码

以上的一切操作都是发生在本地，为了能够发送pull request，你需要将本地的改动提
交到github。

``` bash
git push origin <working-branch-name>
```

 到这里我们的一个工作流程就完成了，如果我们有一个CI系统，我们可以选择我们提
交的这个分支跑一下CI，确保不会让我们的系统挂掉。最后就是发送一条
pull request，这个[官网介绍](https://help.github.com/articles/using-pull-requests)的很详细啦。

最后说明一下为什么要创建分支而不直接在主分支上操作。其实主要是为了保证一个
干净的主分支。随时让我们可以基于这个干净的分支创建新的分支，所有的操作和修
改都是与主分支隔离开的。主分支只负责和upstream进行同步。如果在本地提交了有
问题的代码那还好，大不了就是删掉本地的库，重新再克隆一个。一旦提交到github
上，主分支就污染了，为了保证减少无意义的提交（revert之前的commit）就需要重
新fork了。

[Effective pull requests and other good practices for teams using github](http://codeinthehole.com/writing/pull-requests-and-other-good-practices-for-teams-using-github/)
