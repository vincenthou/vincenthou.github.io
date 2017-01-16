![image](http://bower.jsbin.cn/img/bower-logo.png)
[Bower](http://bower.io/) 是 twitter 推出的一款包管理工具。我们平时开发过程中不可能所有的功能都是我们白手起家实现的，有一些功能已经有了成熟的解决方案，那就是拿来主义了。作为后端开发来讲，ruby有gem，nodejs有npm，python有pip，java有maven。我们在前端开发时需要一些常用的框架和工具库，例如jqeury,  jquery-ui, underscore,  backbone等等。想想每次我们需要添加这些库的时候是怎么做的吧。

现在你不需要在四处寻找某个版本的文件，下载并且拷贝到本地了，只需要一条命令就可以搞定

``` bash
bower install jquery
```
# 开始安装

首先确保你已经安装了nodejs和npm包管理工具，最好你的机器上也安装了Git，有些bower以来的包需要通过她来下载安装，在bash中执行命令

``` bash
npm install -g bower
```

这样就OK了，详细的命令可以查看bower help

``` bash
Usage:

    bower <command> [<args>] [<options>]

Commands:

    cache                   Manage bower cache
    help                    Display help information about Bower
    home                    Opens a package homepage into your favorite browser
    info                    Info of a particular package
    init                    Interactively create a bower.json file
    install                 Install a package locally
    link                    Symlink a package folder
    list                    List local packages
    lookup                  Look up a package URL by name
    prune                   Removes local extraneous packages
    register                Register a package
    search                  Search for a package by name
    update                  Update a local package
    uninstall               Remove a local package

Options:

    -f, --force             Makes various commands more forceful
    -j, --json              Output consumable JSON
    -l, --log-level         What level of logs to report
    -o, --offline           Do not hit the network
    -q, --quiet             Only output important information
    -s, --silent            Do not output anything, besides errors
    -V, --verbose           Makes output more verbose
    --allow-root            Allows running commands as root

See 'bower help <command>' for more information on a specific command.
```
# 开始使用

打开bash，进入你的项目目录，执行命令

``` bash
bower install jquery
```

一个新的文件夹bower_components会被创建出来，你使用bower安装的所有包都会放在这个目录下面，**一般不要直接修改这个目录下的文件**。
Bower 在下载的时候会去 server 上找名字对应的 git 库，下载后切换到对应的版本，如果未指定则是最新的。
除了通过以上别名的方式，还可以指定github上的代码库，某个文件或者压缩包，本地的Git仓库来进行安装，你还可以指定安装的版本

``` bash
bower install git://github.com/someone/some-package.git
bower install someone/some-package
bower install <package>#<version>
```
- 通过 bower list 可以看到本地安装的所有的包。
- 如果想要搜索你需要的包何以使用 bower search 命令或者在[这里](http://sindresorhus.com/bower-components)看看。
- 另外每次你在线安装后bower都会产生缓存

``` bash
bower cache list
```

通过这查看缓存，你可以知道你曾经下载过哪些文件，你也可以通过缓存来安装包

``` bash
bower install <package> --offline
```
