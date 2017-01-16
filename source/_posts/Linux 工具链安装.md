最近换用ubuntu的系统，很多工具要安装，挺繁琐的，在这里记录下来
# LNMP Setup

``` sh
sudo apt-get update
```
## Nginx

``` sh
sudo apt-get intsall nginx
```
- 所有的配置文件都在/etc/nginx下,并且每个虚拟主机已经安排在了/etc/nginx/sites-available下
- 程序文件在/usr/sbin/nginx \* 日志放在了/var/log/nginx中
- 在/etc/init.d/下创建了启动脚本nginx
- 默认的虚拟主机的目录设置在了/var/www/nginx-default

``` sh
sudo /etc/init.d/nginx start 
    or
service nginx start

sudo killall apache2 
```
## PHP

``` sh
sudo apt-get install php5 php5-cgi php5-mysql php5-curl php5-gd php5-idn php-pear php5-imagick php5-imap php5-mcrypt php5-memcache php5-mhash php5-ming php5-pspell php5-recode php5-snmp php5-tidy php5-xmlrpc php5-sqlite php5-xsl
```
### spawn-fcgi

为什么要安装spawn-fcgi呢,它用来控制php-cgi进程,以防止进程崩溃或是单进程的效率太低.
网上很多人都说要使用spawn-fcgi必须得安装lighttpd,实际上不必要,可以直接安装spawn-fcgi

``` sh
sudo apt-get install spawn-fcgi 
```
## MySQL

``` sh
sudo apt-get install mysql-server mysql-client
sudo /etc/init.d/mysql start
```
## Configuration
1. 配置根目录和绝对路径

``` sh
sudo vi /etc/nginx/sites-available/default
```

修改配置文件

``` sh
server {
   ...
root {web_root}
   ...
location ~ .php$ {
    fastcgi_pass 127.0.0.1:9000;
    fastcgi_index index.php;
    fastcgi_param SCRIPT_FILENAME /var/www/nginx-default$fastcgi_script_name;(Optional)
    include /etc/nginx/fastcgi_params; 
}
```

去掉注释/etc/php5/cgi/php.ini中cgi.fix_pathinfo=1;这样php-cgi方能正常使用SCRIPT_FILENAME这个变量.
2. 启动fastcgi并且重新载入nginx配置

``` sh
sudo /etc/init.d/nginx reload
sudo /usr/bin/spawn-fcgi -a 127.0.0.1 -p 9000 -C 5 -u www-data -g www-data -f /usr/bin/php5-cgi -P /var/run/fastcgi-php.pid
```

参数含义如下
- -f 指定调用FastCGI的进程的执行程序位置,根据系统上所装的PHP的情况具体设置
- -a 绑定到地址addr
- -p 绑定到端口port
- -s 绑定到unix socket的路径path
- -C 指定产生的FastCGI的进程数,默认为5(仅用于PHP)
- -P指定产生的进程的PID文件路径
- -u和-g FastCGI使用什么身份(-u 用户 -g 用户组)运行,Ubuntu下可以使用www-data,其他的根据情况配置,如nobody、apache等现在可以在web根目录下放个探针或php文件测试一下了

设置开机启动

``` sh
sudo vi /etc/rc.local
```

添加一行   

``` sh
/usr/bin/spawn-fcgi -a 127.0.0.1 -p 9000 -C 5 -u www-data -g www-data -f /usr/bin/php5-cgi -P /var/run/fastcgi-php.pid
```
# Sublime Text
## Setup

``` sh
sudo add-apt-repository ppa:webupd8team/sublime-text-2  
sudo apt-get update 
sudo apt-get install sublime-text (sublime-text-dev/sublime-text-2)
```
## Plugins
### Package Control

``` python
import urllib2,os; pf='Package Control.sublime-package'; ipp=sublime.installed_packages_path(); os.makedirs(ipp) if not os.path.exists(ipp) else None; urllib2.install_opener(urllib2.build_opener(urllib2.ProxyHandler())); open(os.path.join(ipp,pf),'wb').write(urllib2.urlopen('http://sublime.wbond.net/'+pf.replace(' ','%20')).read()); print 'Please restart Sublime Text to finish installation'  
```
- Emmet - CSS style HTML Coding [Manually install pyv8](https://github.com/emmetio/pyv8-binaries)
- Javascript Refactor - Refactor javascript code tool
- DocBlockr - Generate comment pattern based on your code
- MarkdownEditing - Visual editing markdown document
- Snippets - A lot of snippets for code block generation (AngularJS Jade LESS Sass SCSS CSS Grunt Gulp Mocha PHPUnit)
- Terminal - Open terminal in the sublime text project folders
- jsFormat - Format JS file
- CanIUse - Check the compability of CSS property
- Themes - Theme Brogrammer
