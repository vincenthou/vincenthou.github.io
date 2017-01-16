当我们完成了一个网站的开发工作后，为了保证网站的服务质量一般都会进行压力测试，有很多开源的工具可供使用，每种压力测试工具都提供了很好的特性支持，例如大名鼎鼎的JMeter。如果我们只是想要看到某个页面的效果，或者在几个技术方案中作出权衡，有个很好用的工具apache自带的压力测试工具ab(就是apache benchmark的缩写啦)
# ubuntu上安装

``` sh
sudo apt-get install apache2-utils
```
# 工具参数

ab 有许多配置参数，就不一一列出，可以通过help指令或者参考资料了解，说一个常用的参数

``` sh
- n 总共需要发出的请求数
- c 发送请求使用的并发数（简单理解为客户端数量）
- t 等待服务器的最大响应时间（过期不候）
- k 使用HTTP的keep-alive特性 (这个需要服务器的支持)
```

[keep-alive特性](http://blog.sina.com.cn/s/blog_564fc50a0100n7r6.html)
# 结果分析

贴出一些使用ab压力测试的结果:
- [压测nginx swoole node.js golang的http server](https://github.com/matyhtf/swoole/blob/master/wiki/bench.md)
- [多核单服务器各种配置和业务压力下的node.js性能测试  ](http://snoopyxdy.blog.163.com/blog/static/6011744020117315192204/)

测试结果中有很多有用的信息，我们一般比较关注两个指标：
Request per second 每秒处理请求数（服务器负载能力的有力体现）
Time per request 每个请求的响应时间 （注意这里有两个值）
由于对于并发请求，cpu实际上并不是同时处理的，而是按照每个请求获得的时间片逐个轮转处理的，所以基本上第一个Time per request时间约等于第二个Time per request时间乘以并发请求数
# 小试验
- 基准测试

``` sh
ab -c 1000 -n 1000 http://localhost:8001/
ab -c 1000 -n 1000 http://localhost/index.html
ab -c 1000 -n 1000 http://localhost/index.php
```

[nodejs官网](http://nodejs.org/) hello world: RPS 2000左右(实验次数越多，效率急剧下降)
ngnix 默认配置加载静态文件：RPS 10000左右
ngnix 默认配置加载php文件：RPS 5000左右
ngnix 反向代理nodejs: RPS 7000左右
ngnix 反向代理加负载均衡2个nodejs进程: RPS 8000左右
- 减小并发数

``` sh
ab -c 100 -n 1000 http://localhost:8001/
ab -c 100 -n 1000 http://localhost/index.html
ab -c 100 -n 1000 http://localhost/index.php
```

[nodejs官网](http://nodejs.org/) hello world: RPS 4000左右
ngnix 默认配置加载静态文件：RPS 10000左右 (添加-k选项可以达到16000)
ngnix 默认配置加载php文件：RPS 500左右
ngnix 反向代理nodejs: RPS 3000左右
ngnix 反向代理加负载均衡2个nodejs进程: RPS 2700左右
- 增加请求量

``` sh
ab -c 1000 -n 2000 http://localhost:8001/
ab -c 1000 -n 2000 http://localhost/index.html
ab -c 1000 -n 2000 http://localhost/index.php
```

[nodejs官网](http://nodejs.org/) hello world: RPS 2000左右(实验次数越多，效率急剧下降)
ngnix 默认配置加载静态文件：RPS  8000左右(添加-k选项可以达到12000)
ngnix 默认配置加载php文件：Connection reset
ngnix 反向代理nodejs: RPS 2000左右
ngnix 反向代理加负载均衡2个nodejs进程: RPS 4000左右（但是不是很稳定）
- 结论
1. nodejs 处理高并发请求效率较差，并发数越多稳定性越不好
2. nginx适合作为静态文件服务器，设置keep-alive header可以有效提升服务性能
3. nginx使用负载均衡和反向代理可以有效提高nodejs处理并发的能力
4. nodejs 处理低并发的大量请求效率还是很不错的，适合作为rest API
5. 简单使用php+nginx效率处理大量低并发请求明显小于nodejs, 大并发时效果出奇地好（有点疑问![疑问](http://mat1.gtimg.com/www/mb/images/face/32.gif)）
# 参考

[使用ab对nginx进行压力测试](http://www.nginx.cn/110.html)
[Apache ab 压力测试](http://leepiao.blog.163.com/blog/static/485031302010234352282/)
