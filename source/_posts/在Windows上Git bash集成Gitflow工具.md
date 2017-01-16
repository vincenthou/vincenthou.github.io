Git作为版本控制工具越来越流行，但是考虑到Git使用的灵活性，在进行大型项目开发的时候遵循一定的规范还是很有必要的，比较实用的一种工作流程要数[Gitflow](https://www.atlassian.com/git/workflows#!workflow-gitflow)了，刚开始使用的时候会感觉这个流程很复杂，为了简化操作，Gitflow的工具应运而生，下面简单介绍一下。
## 在Windows上使用Gitflow规范流程
### 1. 将Gitflow克隆到本地

``` bash
git clone -recursive git://github.com/nvie/gitflow.git
```
### 2. 下载与msysgit集成Bin文件

[util-linux-ng-2.14.1-bin](http://sourceforge.net/projects/gnuwin32/files/util-linux/2.14.1/util-linux-ng-2.14.1-bin.zip/download)

[util-linux-ng-2.14.1-dep](http://sourceforge.net/projects/gnuwin32/files/util-linux/2.14.1/util-linux-ng-2.14.1-dep.zip/download)
### 3. 将Bin文件拷贝到msysgit的bin目录下

下载完成后分别将两个压缩包解压文件下bin目录中的getopt.exe，libintl3.dll和libiconv2.dll文件拷贝到msysgit安装目录下的bin目录中
### 4. 安装Gitflow工具

打开cmd命令行，进入已经克隆好的gitflow contrib目录中执行命令

``` bash
D:\gitflow contrib>msysgit-install.cmd "C:\Program Files\Git".
```
### 5. 开始在console中使用Gitflow

``` bash
usage: git flow <subcommand>

Available subcommands are:
   init      Initialize a new git repo with support for the branching model.
   feature   Manage your feature branches.
   release   Manage your release branches.
   hotfix    Manage your hotfix branches.
   support   Manage your support branches.
   version   Shows version information.

Try 'git flow <subcommand> help' for details.
```
## 使用Gitflow之后的开发流程
### 1. 为本地代码库添加Git flow特性

``` bash
git flow init
```

使用默认值进行初始化设定
### 2. 开始增加新特性/修复bug

``` bash
git flow feature start <feature-branch-name>
```
### 3. 完成开发的工作

修改一些文件，添加修改的文件到暂存区(staging area)

``` bash
git add .
```

将修改后的文件提交到本地的版本库中

``` bash
git commit -am 'Add a new feature'
```
### 4. 完成新特性/修复bug

``` bash
git flow feature finish <feature-branch-name>
```
### 5. 同步到中央代码库

``` bash
git flow feature publish <feature-branch-name>
```
## 参考

[详细的Gitflow使用方法参考](http://danielkummer.github.io/git-flow-cheatsheet/index.zh_CN.html)

[Gitflow原型](http://nvie.com/posts/a-successful-git-branching-model/)
[TortoiseGit日常使用指南](http://wenku.baidu.com/view/1ea291bf960590c69ec37694.html)
