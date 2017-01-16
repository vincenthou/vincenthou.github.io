# 背景

之前对于JS实现继承的机制有一些粗浅的认识，可以简单实现case by case的继承 #16 ，继承的实现是没有复用的，最近专门研究了下，实现一个通用的继承实现
# 经典继承的实现和改进
## 直接使用原型链

``` javascript
function inherit(Sub, Super) {
    Sub.prototype = new Super();
}
```

这种实现是最简单直接的，但是这样生成的Sub实例在复用Super的属性和方法的时候都是基于原型链查找的，子类如果没有覆盖父类的方法或者属性，所有子类实例都是共享的同一份数据，对于方法这样挺好的，但是对于属性就不行了，特别是对于数组和引用类型的属性（除非重新赋值，不然所有实例引用的是同一份数据）。单纯这样写的继承方式是没有办法自定义构造方法的。
## 借用构造函数

``` javascript
function Super(name) {
    this.arr = arr;
}

function Sub(arr, age) {
    this.age = age;
    Super.call(this, arr);
}
```

为了保证子类会实现父类属性的深拷贝，在子类的构造函数里面显示调用父类的构造函数，通过向call或者apply函数传递this参数绑定添加属性的上下文。
## 共享原型

通常的做法是上面两种做法的结合，但是这样又会有另外一个问题，那就是两次调用构造函数，这是第一次

``` javascript
Sub.prototype = new Super();
```

为了实现原型链，实例化父类对象赋值为Sub的原型，完成一次父类构造含函数调用。
这是第二次

``` javascript
Super.call(this, arr);
```

为了深拷贝父类的属性，完成二次父类构造含函数调用。
既然实例属性复用可以通过借用构造函数，我们其实只是想要获得所有父类原型上的属性或者方法，可以通过共享原型实现替换原来的实现

``` javascript
function inherit(Sub, Super) {
    Sub.prototype = Super.prototype;
}
```
## 增加代理构造函数

共享原型可以方便的让子类使用父类的原型属性和方法了，但是仍然存在问题。因为子类原型是直接指向父类原型的，这种引用类型的赋值倒置，所有继承父类的子类原型是一样的。在完成子类的继承之后，如果修改任何子类的原型，这份修改会影响到所有的其他子类原型和父类。既然不能直接引用父类原型，那就添加一个代理层就好了

``` javascript
function inherit(Sub, Super) {
    var F = function(){};
    F.prototype = Super.prototype;
    Sub.prototype = new F();
}
```

这里定义的这个F就是我们说的代理构造函数，因为是单独实例化了代理构造函数( new F() )，这样之后对于子类原型的修改都是单独的，不会相互影响，更不会影响到父类原型。
## 存储父类原型和重置构造函数指针

``` javascript
function inherit(Sub, Super) {
    var F = function(){};
    F.prototype = Super.prototype;
    Sub.prototype = new F();
    Sub.uber = Super.prototype;
    Sub.prototype.construtor = Sub;
}
```

因为添加了代理层，子类和父类的关系不是很明朗，添加一个uber属性标示父类原型。我们如果察看Sub的实例构造器( sub.constructor )，发现结果是Super，为了弥补这一不合理的现象，重置constructor是一个不错的方法。
## 优化继承的实现

其实这里的代理构造函数是可以复用的，不需要每次继承父类的时候都创建一个，通过创建闭包，可以让所有的子类继承都使用，同时由于这里对于子类原型的复制都是实例化一个新的代理实例，不存在原型共享的问题。

``` javascript
var inherit = (function(){
    var F = function() {};
    return function(Sub, Super) {
        F.prototype = Super.prototype;
        Sub.prototype = new F();
        Sub.uber = Super.prototype;
        Sub.prototype.constructor = Sub;
    };
})();
```

搭配借用构造函数为子类属性赋值，这已经是最完美的解决方案了。顺便提一下，这里有一个经典的原型继承模式（我觉得是基于原型链的浅拷贝）

``` javascript
function object(o) {
    var F = function(){};
    F.prototype = o;
    return new F();
}

var obj = {name:'vincent'};
var cloned = object(obj);
obj.name = 'test';
console.log(cloned.name);//output 'test'
```

在ECMAScript 5中通过[Object.create](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/create)实现了这一行为，在现代浏览器和nodejs中可以直接调用。
## 继承静态属性和方法

上面的解决方案已经能够解决我们处理JS面向对象继承的问题了，进一步来说。我们的继承概念需要能够继承父类的静态属性和方法，再添加点修改

``` javascript
var inherit = (function(){
    var F = function() {};
    return function(Sub, Super) {
        F.prototype = Super.prototype;
        Sub.prototype = new F();
        Sub.uber = Super.prototype;
        Sub.prototype.constructor = Sub;
        //add static function from parent class
        var prop;
        for (prop in Super) {
            !Sub[prop] && (Sub[prop] = Super[prop]);
        }
    };
})();
```
# 复制属性的继承方式
## 简单属性复制

我们有的时候不需要进行完成经典的继承，可能只是继承，或者说是拷贝一些属性（基于对象），通过简单的属性循环就可以实现，hasOwnProperty会排除掉原型上的属性。

``` javascript
function extend(parent, child) {
    var prop;
    child = child || {};
    for (prop in parent) {
        if (parent.hasOwnProperty(prop)) {
            child[prop] = parent[prop];
        }
    }
    return child;
}
```
## 深度复制

上面的实现其实对于对象的拷贝来说其实只是实现了简单的浅拷贝，对于引用类型和数组类型会有潜在的问题，我们可能需要一个更加健壮的方法实现。

``` javascript
function extend(parent, child, deep) {
    var prop;
    child = child || {};
    var aType = '[object Array]';
    for (prop in parent) {
        if (parent.hasOwnProperty(prop)) {
            //check if the value is array or object
            if (deep && 'object' === typeof parent[prop]) {
                child[prop] = aType ===  {}.toString.call(parent[prop]) ? [] : {};
                //recursive extend array and object value
                extend(parent[prop], child[prop], true);
            } else {
                child[prop] = parent[prop];
            }
        }
    }
    return child;
}
```

通过添加第三个参数来确定是否进行深拷贝，这里深拷贝的实现就是在遍历属性的时候判断属性值是否是引用类型或者数组类型，如果是的话继续进行深拷贝，知道拷贝到普通类型值为止。
## 混入

在JS中通过借用多个构造函数，可以实现多继承（基于构造函数）

``` javascript
function Super1(){}
function Super2(){}
function Super3(){}
function Sub() {
    Super1.call(this)
    Super2.call(this)
    Super3.call(this)
}
```

我们也可以使用一个更灵活的方法来实现多继承（基于对象），这就是混入，我们可以通过这个方法方便的进行配置参数的合并

``` javascript
function mixin() {
    var prop, mixed = {};
    for (var i = 0, len = arguments.length; i < len; i++) {
        for (prop in arguments[i]) {
            console.log(mixed)
            if (arguments[i].hasOwnProperty(prop)) {
                mixed[prop] = arguments[i][prop];
            }
        }
    }
    return mixed;
}
```

Jquery中是通过extend方法来实现的mixin，并且是以上两个功能的合并（同时支持深拷贝和多继承），深拷贝的标示变量是第一个参数，详细的[Jquery实现参考](https://github.com/jquery/jquery/blob/master/src/core.js#L124)
# 参考

[Classical Inheritance in JavaScript](http://www.crockford.com/javascript/inheritance.html)
[OOP in JS, Part 2 : Inheritance](http://phrogz.net/JS/classes/OOPinJS2.html)
[Inheritance and the prototype chain](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Inheritance_and_the_prototype_chain)
