# 背景

随着前端开发的复杂性不断提升，规模逐渐增大，OOP的价值就凸显出来了，在开发nodejs应用时这方面的知识更是必不可少的，这里总结一些自己的认识。
# 对象创建

传统面向对象语言是通过类来实现的，类就像一个模板，你定义好模板的样子，就可以用这个模子压制(new)出来你想要的产品了。javascript是基于原型的，我理解就是依葫芦画瓢，所有对象是基于原型的拷贝（浅拷贝，只是复制值，对于引用类型会有操作同一份数据的问题）。这样的差别造就了javascript对象创建的特殊性。
## 单例对象

最简单的对象创建方法莫过于使用对象字面量了

``` javascript
var obj = {
    name: 'vincent',
    gender: 'male',
    say: function() {
        console.log('Hi');
    }
}
```

这样太暴露了，于是有了工厂方法

``` javascript
function createObj() {
     return  {
        name: 'vincent',
        gender: 'male',
        say: function() {
            console.log('Hi');
        }
    }
}
```

其实这样没有实现隐藏，返回的对象上的属性和方法是完全对外可以访问的，可以使用_name这样的写法来表明该属性是私有的，通过闭包可以有效解决这个问题，这也被称为稳妥的构造函数模式

``` javascript
function createObj() {
     var name = 'vincent';
     var gender = 'male';
     var sayGender = function() {
           console.log('My gender is ' + gender);
     }
     var say = function() {            
            console.log('Hi, I am' + name);
            sayGender();
     };
     return  {
        say: say
    }
}
```
## 构造函数

单纯使用构造函数和单纯基于原型都会有问题，最好的方式就是结合两者之所长，在构造函数中添加属性，在构造函数原型上添加多个实例共享的方法

``` javascript
function Person(name, gender) {
    this.name = name;
    this.gender = gender;
}
Person.prototype = {
    constructor: Person, //optional
    say: function() {
        console.log('I am ' + this.name);
    }
}
```

我们还可以将原型对象的动态添加放在构造函数里面，这样就有了点后端定义类的感觉了

``` javascript
function Person(name, gender) {
    this.name = name;
    this.gender = gender;
    if ('function' === typeof this.say) {
        Person.prototype.say = function() {
            console.log('I am ' + this.name);
        };
        ...
    }
}
```

有时候我们需要扩展某个对象，但是又不希望在原来对象的原型上添加，就可以这样来做了

``` javascript
function MyDate() {
    var date = new Date();
    date.now = function() {
        return this.getTime();
    }
    return date;
}
var date = new MyDate();
console.log(date.now());
```

这种方式有点像是一个工厂方法，我们自定义的方法在每个工厂产生的实例上添加，因为有返回值，使用MyDate的时候new关键字是可选的
# 对象继承
## 原型链继承

javascript本身没有继承的直接支持，要想实现面向对象思想中继承的特性，就需要些额外的工作了。由于每个对象都会有一个**proto**属性指向自己的原型，而原型链的特性又和作用域链的特性类似（当前scope找不到就向上查找），我们可以通过原型链来模拟简单继承

``` javascript
function Super(prop) {
    this.prop = prop;
}
Super.prototype.superFn = function() {}
function Sub() {}
Sub.prototype = new Super();
```
## 借用构造方法

这样子类实例就会通过原型链查找访问到父类的所有属性和方法，看起来不错，但是原型浅拷贝带来的问题，这样的方法还有待改进。要想实现子类的深拷贝，调用一下父类的构造方法就好了

``` javascript
function Super(prop) {
    this.prop = prop;
}
Super.prototype.superFn = function() {}
function Sub() {
    Super.call(this, 'hi');
}
```
## 组合继承

这样属性倒是深拷贝了，同时也可以很方便控制给父类构造函数传递什么样的参数，但是父类的方法就用不了了，组合的方式是一个不错的解决方案

``` javascript
function Super(prop) {
    this.prop = prop;
}
Super.prototype.superFn = function() {}
function Sub() {
    Super.call(this, 'hi');
}
Sub.prototype = new Super();
```
## 寄生组合继承

同时照顾到了属性和方法，就像创建对象的构造函数模式一样。这样已经很不错了，不过我们进一步优化，还是能找到问题的。很明显这里Super构造函数被重复调用了两次，一个在子类原型复制时，一个在子类构造函数中。其实我们调用 Sub.prototype = new Super() 的意义在于创建子类关联父类的原型链，父类属性的添加可以通过调用 Super.call(this) 来实现，这就是我们的优化点

``` javascript
function Super(prop) {
    this.prop = prop;
}
Super.prototype.superFn = function() {}
function Sub() {
    Super.call(this, 'hi');
}
Sub.prototype = Super.prototype;
Sub.prototype.constructor = Sub; //保证构造函数检测正确
var instance = new Sub();
console.log(instance.constructor === Sub);
```
# 对象mixin

以上继承的实现只能实现单继承，如果我们希望动态的多个属性就需要借用mixin的概念，使用过ruby的开发对这个概念不会感到陌生，mixin比纯粹的继承组织模式要灵活很多，下面是一个简单的实现

``` javascript
Object.extend = function (dest, src) {
    for (prop in src) {
          if (src.hasOwnProperty(prop)) {
              dest[prop] = src[prop];
         }
    }
  return dest;
};
```

[jquery的实现](https://github.com/jquery/jquery/blob/master/src/core.js#L124)，当然jquery为了实现插件的机制，做了一些额外的事情
# 参考
- [Javascript 面向对象编程](http://coolshell.cn/articles/6441.html)
- [面向对象编程之概论](http://www.cnblogs.com/TomXu/archive/2012/02/03/2330295.html)
- [面向对象编程之ECMAScript实现（推荐）](http://www.cnblogs.com/TomXu/archive/2012/02/06/2330609.html)
- [不会JS中的OOP，你也太菜了吧](http://www.cnblogs.com/JChen666/p/3375894.html)
- [Jquery core](https://github.com/jquery/jquery/blob/master/src/core.js)
