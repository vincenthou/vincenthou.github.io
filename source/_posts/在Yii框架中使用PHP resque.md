总体上来说PHP resque做的事情很简单，就是创建后台任务队列，让守护进程去“消费”执行它们，PHP resque为我们做了很多实现的工作，只需要简单的配置，编写我们需要执行的任务代码就可以了。
[Resque思想的介绍](https://github.com/blog/542-introducing-resque)

因为[php-resque](https://github.com/chrisboulton/php-resque)使用了php的PCNTL函数，所以只能在Linux下运行，下面的安装和说明默认都是在Linux平台下的。
**Tip**:
1. 安装的时候要保证clone下载php resque的文件夹是可写的，chmod一下
2. 修改demo中的resque.php文件，改为"require **DIR**.'/../bin/resque'",必须加上当前文件的绝对路径，否则不好使，bin目录下方的文件是resque，没有php后缀名的，坑爹啊
3. xdebug启用: display_errors=On
4. 为composer工具设置别名，方便使用

``` bash
alias composer='/usr/local/bin/composer.phar'
```

简单实验一下，就有个感性的认识了，下面开始正式的工作。
补充：基于开源项目yii-resque开发的项目，添加了一些特性，修复原有代码中的一些bug（原作者好久不维护了，bug真心不少），和同事一起搞出来的[yii-resque-ex](http://git.oschina.net/VincentHou/yii-resque-ex)
## 参考

安装和简单的Demo实验
- [http://avnpc.com/pages/run-background-task-by-php-resque](http://avnpc.com/pages/run-background-task-by-php-resque)
- [(http://www.zrwm.com/?p=4464)](http://www.zrwm.com/?p=4464)

进一步的介绍
- [http://blog.hsatac.net/2012/01/php-resque-introduction/](http://blog.hsatac.net/2012/01/php-resque-introduction/)
- [http://docs.dotcloud.com/tutorials/php/resque/](http://docs.dotcloud.com/tutorials/php/resque/)
- [Best practice](http://stackoverflow.com/questions/11814445/what-is-the-proper-way-to-setup-and-use-php-resque)

php-resque的扩展：
- [自动根据Job的数量计算需要启动的Worker数](http://blog.hsatac.net/2012/02/php-resque-auto-scale-workers/)
- [PHP Resque Pool](https://github.com/ebernhardson/php-resque-pool)

Web Monitor
- [ResqueBoard](http://resqueboard.kamisama.me/)
- [Resque-web](https://github.com/defunkt/resque-web)

Bash Monitor
- [Supervisord](https://github.com/chrisboulton/php-resque/issues/32)
- [Supervisord 配置](http://www.54chen.com/_linux_/supervisord-manage-service.html)
