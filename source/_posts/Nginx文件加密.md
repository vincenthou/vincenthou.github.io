# 背景

我们有时候需要放置一些静态文件资源在服务器上方便团队成员共享，但是又不希望这些资源对外可以访问，apache上提供了方便的配置方法实现，nginx上面也有类似的功能，今天配置了下，本来以为挺简单的，还颇费了些周折，记录一下
# 配置Nginx

Nginx有个auth_basic的模块，直接看配置

``` sh
location ^~ /protected/ {
    # First attempt to serve request as file, then
    # as directory, then fall back to index.html
    try_files $uri $uri/ /index.html;
    # Uncomment to enable naxsi on this location
    # include /etc/nginx/naxsi.rules
    auth_basic "vincent";
    auth_basic_user_file /etc/nginx/conf.d/htpasswd.pw;
}
```

主要是最后的两行配置，auth_basic定义弹出输入用户名和密码提示信息，auth_basic_user_file代表导入的用户名和密码列表配置文件，这样配置以后所有访问protected目录下文件的请求都需要用户输入用户名和密码才能登陆。
# 创建用户名和密码列表文件

根据上面的配置创建列表文件

``` sh
cd /etc/nginx/conf.d
sudo touch htpasswd.pw
```

!这里文件可以取任何文件类型，但是千万不要使用.conf后缀，开始文件叫htpasswd.conf，重启nginx一直报语法错误。
# 创建生成加密密码的脚本

在任意目录下面创建perl脚本，保存为gpasswd.pl

``` pl
#!/usr/bin/perl
use strict;
my $pw=$ARGV[0] ;
print crypt($pw,$pw)."\n";
```

为脚本添加可执行权限

``` sh
sudo chmod +x gpasswd.pl
```
# 生成crypt加密的密码

如果我们想要生成的密码是happy, 执行命令

``` sh
./gpasswd.pl happy
```

在命令行里面输出hafbpgNasZSjY，这就是我们要的crypt加密后的密码了。
# 将加密后的密码添加到用户列表

打开之前创建的htpasswd.pw文件，给用户user添加密码happy

``` sh
user:hafbpgNasZSjY
```

重启nginx服务器，一般这样当你再次访问{domain}/protected 下文件的时候就需要输入用户名和密码了。

``` sh
sudo /etc/init.d/nginx reload
```
# 可能的问题
- 有时候会报访问权限的错误，可能是文件夹的读写权限没有配置

``` sh
cd /var/www
sudo chown -R www-data:www-data protected
sudo chmod 755 protected
```
- 如果 不用 ^~ /protected/ 而用 /protected 的话 那么将只能对目录进行验证，直接访问其下的文件，将不会弹出登录验证
# 参考
- [Nginx 目录或 网站加密认证](http://blog.csdn.net/cuiyuan9/article/details/13627359)
- [Apache和nginx对网站目录加密](http://bolg.sinaapp.com/html/2011/1105.html)
- [nginx设置网站密码访问](http://blog.chaorenmao.com/?p=578)
