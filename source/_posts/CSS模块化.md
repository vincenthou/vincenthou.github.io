# 背景

如果只是几个简单页面从快速实现上来说，CSS模块化完全没有必要，想怎么写就怎么写。但是随着业务需求的变更或者是规模的增长，不只是你一个人在写CSS代码，CSS模块化就显得尤为重要了，如果没有很好的组织，到了后期简直是一场噩梦。
# CSS代码组织的理论

对于CSS模块化，前辈已经有了很多探索和实践，这里简述一下：
## 简单的组建化

很直观的当我们拿到设计图的时候，会抽出来一些通用的组件，例如按钮，下拉框，选项卡等，这是很自然的行为，当组建的行为或者样式更改的时候，我们之需要修改相关的CSS片段就好了，这里为了防止一些通用名称如hd(header)，ft(footer)冲突，会需要是有后代选择器来构建命名空间，但是权重的提升会带来潜在的问题，这便是最朴素的做法。
## OOCSS

这里的OO其实不是我们编写代码所理解的OO，其实更准确的理解应该是对于页面元素的抽象。OOCSS有两个最基本的原则就是：
1. 结构和皮肤分离
2. 容器和内容分离
这样区分的目的其实是分离变化，另外我觉的这两条规则隐含的一点是多使用组合来构建组件，少使用后代选择器（后代选择器提升的权重会是维护的梦魇）。其实在设计时为了保持风格统一，很多组件也会有重复的设计样式。
例如gitbhu这个编辑页面上，有个“This repository”和“Unwatch”的button，这两个元素从组件角度来看是不一样的，前者后面紧跟的是一个输入框，而后者后面跟的是操作的统计次数，但是它们又有相同的地方，就是都有圆角（左上和右下）box-shadow和小箭头(点击能有下拉)，这就是两个不同组件的共有特性部分，我们就可以这样来写：

``` css
.select-btn {
    position: relative;
    border-top-left-radius: 2px;
    border-bottom-left-radius: 2px;
   box-shadow: 0 0 5px rgba(81,167,232,0.5);
}
.select-btn:before {
    content: "";
    position: absolute;
    top: 10px;
    right: 10px;
    display: block;
    width: 0;
    height: 0;
    border: 4px solid;
    border-right-color: transparent;
    border-left-color: transparent;
    border-bottom-color: transparent;
}
```

这种做法会带来一定的复杂度，一般情况下，我们无法一下识别出这些相似的特性，或者由于设计师的修改，相似的特性会被转移或者破坏，这就需要不断的重构代码，对于开发的抽象能力有一定的要求。因为使用组合，这种方式会比较灵活。
## SMACSS

可扩展和模块化的CSS架构 (Scalable and Modular Architecture for CSS) 是把所有的CSS文件视为一个整体来考虑的，他的基本理论是将CSS代码分类，包括基础，布局，模块，状态和皮肤这5类，对于细节代码的书写没有过多的限制，后代选择器和ID选择器也是可用的。
- 基础 就是我们平常会用到的CSS reset，这个一般不会经常修改的
- 布局(l-) 页面的大块的布局结构，例如header，content, footer, siebar这些
- 模块(modula name) 放在布局结构中的小组件，就是朴素模块化中的组件概念
- 状态(is-) 组件或者布局元素的伪类效果（:hover :focus :checked）或者自定义状态类(.is-expanded .is-folded)
- 皮肤 定义颜色和字体

bootstrap就是一个典型的实践，bootstrap使用了normalize.less来reset，布局使用了grid.less来让用户自定义进行布局，core css中的button.less，tables.less等和所有的coponent，状态是跟组件关联的没有单独抽出来，bootstrap有个theme文件方便用户定制组件的皮肤。
## DRY CSS

DRY CSS是最新的一项实践，个人感觉他的上手是最容易，复杂程度也是最低的。他有两条重要的原则
1. 将样式从内容分离
2. 通过控制层叠来避免特殊性

通过内容的语义而不是样式来命名，使用ID选择器也没什么大不了的。
之需要三步你就可以开始上手了：
1. 将可以复用的CSS属性分组
2. 根据语义为分组命名
3. 添加相同分组样式的选择器

![DRY](http://media.creativebloq.futurecdn.net/sites/creativebloq.com/files/images/2013/09/drycss.jpg)
这样写代码会很快，也会有维护的便捷，但是可读性就一般了，比较适合快速搭建原型或者设计搞已经固定下来的开发。
# 额外的工具

近年来CSS预处理器渐渐流行起来，包括LESS，SASS，Stylus这些工具也有了应用。其实总体看来这些DSL的语法特性差别不是很大，包括变量、方法、嵌套和混入等，详细的语法对比看[这里](http://www.oschina.net/question/12_44255)。如果使用ruby作为开发SASS应该是不二的选择，配合Compass更能显出其威力，对应的LESS也有对应国产的[veryless](https://github.com/feichang/veryless)。stylus比较新，语法更自由些（你甚至可以不用括号和冒号），不过个人感觉过分自由的语法不利于项目的统一（总是看起来怪怪的）。虽然bootstrap有SASS和stylus的port，但是官方还是用LESS来开发的，如果是bootstrap的项目还是还是使用LESS比较方便。另外SASS和stylus需要额外的环境配置和工具安装，LESS在开发阶段之需要引入一个js就可以开始工作了，后期可以再编译合并进行优化。虽然都是DRY的实践，有一个语法特性是值得商榷的，就是mixin。mixin在ruby语法中是一个很好的特性，可以方便的实现多重继承，这些CSS预处理器引入了这一个编程语言特性，本来是未可厚非的，但是我觉得不应该过度使用。mixin的好处不用多说，开发时应用起来也很方便，但是一旦过度使用在编译后，CSS中就会有大量重复的代码块。我觉得组合的思想还是更适合在CSS中使用的，在bootstrap的LESS代码中也有体现，无参数的mixin用的是很少的，大部分mixin是使用有参数的（编译后的值不一样），不会造成重复。
# 总结

实际使用CSS预编译器会有一定的学习成本，团队中推动会有阻力（如果大家都有尝试新技术的兴趣，不妨一用），相比之下模块化组织CSS代码的思路会更实用些，根据实际项目需求取舍吧，最适合的才是最好的。
我觉得整体的CSS的模块化组织应该是这样的：
- 使用SMACSS组织整体项目代码
- base: 尽量使用normal.css来作为base，如果要兼具语义化和设计图还原还是自己写
- layout: 可以认为header、footer这些也是module，layout中只是对于这些module在不同结构中的尺寸和布局（保证module的独立性）
- module: 应用OOCSS的思想，使用组合的类来创建组件，模块内使用百分比给定尺寸
- state: module特有的state直接和module相关代码放在一起，通用的state除了一些伪类的效果还有trasition和animation
- theme: 定义常用字体、字号和颜色，方便组合应用到模块和元素上
- common: 借鉴[quick layout](https://github.com/zhangxinxu/quickLayout)的思路，提取出常用的原子类方便组合。另外，清除浮动，垂直居中，字体溢出处理等元素处理的通用解决方案
  最好的情况是这些部分由一两个人来维护，保证风格和意义的统一，其他开发之需要像使用bootstrap一样拿来就用就好。以往的开发过程中按照页面自己写自己的CSS虽然可以很方便的分配人物，但是这样的结果是写出来的代码既有重复，命名冲突也很严重。当然这种做法是颇费周折的，使用DRY CSS的做法就没这么麻烦了，就是出来的代码结构没有这么好看，不注意的话，可能会有很多重复的地方。
# 参考
- [OOCSS](http://oocss.org/)
- [SMACSS](http://smacss.com/)
- [DRY CSS](http://www.vanseodesign.com/css/dry-principles/)
- [4 ways to create CSS that's modular and scalable](http://www.creativebloq.com/css3/create-modular-and-scalable-css-9134351)
- [LESS文档](http://lesscss.net/article/home.html)
- [SASS用法指南](http://www.ruanyifeng.com/blog/2012/06/sass.html)
- [Compass官网](http://compass-style.org/)
- [Compass用法指南](http://www.ruanyifeng.com/blog/2012/11/compass.html)
- [Stylus-NodeJS下构建更富表现力/动态/健壮的CSS](http://www.zhangxinxu.com/wordpress/2012/06/stylus-nodejs-expressive-dynamic-robust-css/)
- [stylus官网](http://learnboost.github.io/stylus/)
