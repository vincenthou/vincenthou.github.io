最近前端圈最火的事情莫过于yarn的横空出世了，从10.11号知道了这个事情看着事情升温和[这个库](https://github.com/yarnpkg/yarn)的star被刷上万，不得不说yarn产品做的好，不如FB广告做的好，有这么一个亲爹背书，这关注度和流行度真的是妥妥的。
# 解决问题

言归正传，这么一个新东西出来肯定是要解决问题的，从官网上宣传上来看，主要集中在这三点:
- **超快:** 下过的包直接从本地拷贝
- **超安全:** 安装一个包之前会先用 checksums 来验证
- **超可靠:** 默认的lock机制，实现一次lock到处安装

其实最有吸引力和解决工程痛点的莫过于速度和可靠性了。一般工程开发都会把node_modules忽略提交(当然出于加速部署的角度，会做出妥协)，每次部署会重新从网络安装所有的包。这些依赖的包由于可能使用了宽泛的版本控制声明(大多数情况还都是这样)，每次重新安装都可能会生成不一样的结果，这种不一样的结果可能是好的结果(修复bug或者引入新特性)，也可能是不好的结果(新的bug或者不向前兼容的改动)。这两点在工程实施中是大家都会遇到的头疼问题，也衍生出来各自的解决方案。
- 我们会把node_modules提交，只有加入新的包或者版本更新的时候才会增量下载，之前安装过的包直接使用版本控制系统中的包。
- 使用个淘宝镜像，使用npm shrinkwrap或者[uber npm-shrinkwrap](https://github.com/uber/npm-shrinkwrap)锁定版本。
- 因为cnpm不支持shrinkwrap，所以不适用于一致性的思路，安装真的很快，但是也有不锁包的观点下面有提到

个人觉得之所以yarn这么收到推崇，从纯工具意义上来说是因为它集中解决了这两个痛点，提供出来简单易用的命令，不用你操心这些细节。同时这个工具带来的意义我感觉可以用函数式编程中的纯函数的概念来表达，就是同样的输入可以高效的产生一致性的输出。看起来一切都很好，可是真的是这样吗？
# 不一样的声音

首先从快这个角度来看，单纯比较yarn和npm，yarn是真的块，没有开发上的历史包袱，利用缓存的效用没得说。然而，有个叫[cnpm](https://github.com/cnpm/cnpm)的工具被默默的忽略了，yarn跟cnpm的安装包速度不论算不算缓存，cnpm都略胜一筹，可以参考下面贺老在知乎上的回答，本地亲测带有编译行为的项目，稳定性上cnpm还更强一点，换了淘宝镜像yarn重新安装几次重试才能成功，而cnpm每次都可以稳定安装而且速度较快。

其次从一致性的角度来看，诚然使用npm shrinkwrap强制要求开发每次加包都同步是不靠谱的，但是我们真的需要完全的强一致性吗？给我们带来问题的其实是非一致性的负面，其实我们有时候也是需要其正面的部分，比如小版本bug修复和新功能增加，这样也不至于我们在需要大版本号升级的时候更加痛苦。消除其负面部分死马提出的观点是加强人工干预，也就是提高依赖包的审核力度，保证依赖链的健康性，当然实际落地需要所有开发者有比较高的素养和判断力，在跨团队情况下也有其局限性和实际操作的尴尬。
# 实际体验
## 安装

从npm迁移成使用yarn，直接用npm安装(保证你的nodejs是较新的版本)

```
npm install -g yarn
yarn --version
```

如果连npm都没有，yarn也提供了各种平台直接的[安装方式](https://yarnpkg.com/en/docs/install)
## 设置镜像

在国内没代理就必须要靠镜像了

```
yarn config set registry 'https://registry.npm.taobao.org'
```
## 常用命令对应关系
- yarn / yarn install -> npm install
- yarn add -> npm install --save
- yarn remove -> npm uninstall --save
- yarn add --dev -> npm install --save-dev
- yarn upgrade -> npm update --save
- yarn global add -> npm install -g
## 一些小优化
- 交互界面更加清新友好，加了很多emoji的icon，报错信息更加清楚
- 提供了yarn why命令查看某个模块被依赖的关系和占用空间信息
- yarn run命令提供了交互式命令方式，在没有指定run哪条命令的时候列出可运行命令让你选择
- yarn ls可以清晰列出项目的依赖关系树
# 结论

yarn结合工程痛点重写了npm的功能并根据实际需要做了扩展，所谓不破不立，积重难返，作为一致性思路的工具真的解决了很多npm的问题，作为不代理环境的安装只是改变了registry，对于依赖于国外源包的编译需求不能很好的解决(也许淘宝会fork一个跟cnpm一样的思路加上各种镜像源)。所以要使用一方面是接受一致性的思路和这种思路下的负面: 忽略小版本优化和bug修复，一方面最好是有代理。由于构建的加速依赖于缓存，如果使用docker部署需要设置好共享的volume。个人项目使用cnpm更方便，结合一些特殊方案消除版本失控影响也可以用于生产环境(提交node_modules或者提高一级依赖包质量)。yarn还处于初期，还有各种问题需要解决，这几天issue就上300多了，还有待发展，只能说前途是光明的，道路是曲折的，在前进中有曲折，在曲折中向前进，前端会越来越好！
# 参考
- [yarn, 不是又一个 npm 第三方客户端](https://zhuanlan.zhihu.com/p/22892675)
- [Facebook推出新的JavaScript模块管理器：Yarn](https://mp.weixin.qq.com/s?__biz=MzA4NjE3MDg4OQ==&mid=2650963709&idx=1&sn=db49ef1793610c178792785bd7d5241f&chksm=843a129bb34d9b8d3316343c60e9b3fd0f6da3d5c36da639d3d3b9f98362497689b413a5211a&mpshare=1&scene=1&srcid=1022htBPlVk3boobjgWA3HZ6&key=c3acc508db720376541159bb6b3a5280a5d4ece0269dd592d76564d31639a178c48858c75c334bb04639b31a5df33cac&ascene=0&uin=MjQ4MjAyNTY0MQ%3D%3D&devicetype=iMac+MacBookAir6%2C2+OSX+OSX+10.10.5+build%2814F27%29&version=11020012&pass_ticket=s2NY74QF%2ByBCzsX7cAwZKBL3Usgfs6BoJilNyFpO7iKndv%2BOjsOS%2BpnRYUfJwpCa)
- [新的 js 包管理工具 yarn 解决了什么问题？](https://zhuanlan.zhihu.com/p/22967139)
- [如何评价Facebook推出的JavaScript模块管理器yarn - 贺师俊的回答](http://www.zhihu.com/question/51502849/answer/126133407)
- [为什么我不使用 shrinkwrap（lock）](https://zhuanlan.zhihu.com/p/22934066)
