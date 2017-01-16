# Angularjs 的一些模式

好久没有整理文章了，这次放个大招 :smile:。使用AngularJS开发应用也有一年的时间，感觉算是比较熟悉了，总想整理些东西出来沉淀一下，但每次想写的时候还是感觉写的深度不够。看了[angularjs-in-patterns](https://github.com/mgechev/angularjs-in-patterns)，感觉正好和自己想整理的东西比较契合，就偷懒翻译一下，只是翻译之余结合项目中的经验，加入了自己的注解。还记得AngularJS的Demo看起来是多么的激动人心，理想很丰满，现实很骨感，真正工程应用起来也是各种的坑啊~~但是AngularJS的很多设计理念和开发思想还是值得借鉴的。也正是这些精心的设计节省了很多的代码，开发效率得到了很大的提升，但有时候原来一些看起来很简单的bug反而变得更加难修。个人感觉AngualrJS还是有他的适用场景的，不是所有项目都适合跟风式地上，CMS在这种模式下可能效果更好些。不过不管怎么说，AngularJS都是组件化的一个很好的思路，DRY在他的核心组件中都有体现。下面闲话少扯，进入正题。

<!--toc-->

## 目录

* [概要](#abstract)
* [介绍](#introduction)
* [AngularJS 概述](#angularjs-overview)
  * [模版](#partials)
  * [控制器](#controllers)
  * [作用域](#scope)
  * [指令](#directives)
  * [过滤器](#filters)
  * [服务](#services)
* [AngularJS 模式](#angularjs-patterns)
  * [服务](#services-1)
    * [单例](#singleton)
    * [工厂方法](#factory-method)
    * [装饰器](#decorator)
    * [门面](#facade)
    * [代理](#proxy)
    * [活动记录](#active-record)
    * [拦截过滤器](#intercepting-filters)
  * [指令](#directives-1)
    * [组合](#composite)
  * [拦截器](#interpreter)
    * [模板视图](#template-view)
  * [作用域](#scope-1)
    * [观察者](#observer)
    * [责任链](#chain-of-responsibilities)
    * [命令](#command)
  * [控制器](#controller-1)
    * [页面控制器](#page-controller)
  * [其他](#others)
    * [模块模式](#module-pattern)
  * [数据映射](#data-mapper)
* [参考](#references)

<!--endtoc-->

## 概要

学习新玩意最好的方法就是看看你已经烂熟于心的东西是怎么在新玩意里面被应用起来的。这篇文档不是要让读者熟悉设计和架构模式，只是想聊聊基本的OOP概念，设计模式和机构模式。
这篇文章的主要是想讲讲不同的软件设计和架构模式是如何在AngularJS和AngularJS的单页应用中应用的。

## 介绍

本文第一章先从AngularJS框架的简单概要说起。这点概要解释了AngularJS的主要组件 - 指令，过滤器，控制器，服务，作用域。第二章的部分描述了在框架中不同的设计和架构模式的实现。这些模式是按照被应用模式的AngularJS组件进行分组的。如果一些模式被多个组件使用，将会有显示的声明。最后一章包含了一些在大多数AngularJS单页应用中被使用的架构模式。

## AngularJS 概述

AngularJS是Google开发的Javascript框架。他就是为了开发基于增删改查的单页应用而生的。SPA是一种网页应用，它只会加载一次，在用户对它进行任何进一步的操作时都不需要重新加载整个页面。这就意味着所有的应用资源（数据，模板，脚本，样式）都要在初始请求载入或者更好 - 按需加载。因为大部分的增删改查应用有共同的特性和需求，AngularJS就是要提供开箱即用的优化工具集。一些重要的AngularJS特性是：

- 双向数据绑定
- 依赖注入
- 关注点分离
- 可测性
- 抽象

关注点分离是通过将每个AngularJS应用拆分成不同的组件，例如：

- 模板
- 控制器
- 指令
- 服务
- 过滤器

这些组件能够在不同的模块内被分组，这样有助于获得更高层次的抽象并且控制复杂度。每个组件封装了应用逻辑的某个特定的逻辑。

**注解：** 单页应用提升了应用的使用体验，但是却提高了开发的复杂度，复杂度过高促使软件可控性下降，bug增多。所以其实单页应用不是银弹，根据需要有针对性的使用才好，控制好可变化的范围和变化时机才能更好利用单页应用的特性。前端应用现在变得越来越复杂，很多原来后端的概念和工作都拿到前端来做，静态资源的管理更多是一个工程问题而不仅仅是个技术问题。对于首次全部加载的方式没有什么特别的，就是对于AMD(requirejs)模块使用R.js压成一个文件，前后端公用CommonJS模块使用browserfiy打包成一个bundle，使用强缓存，数字签名管理性能和部署。这种方式当然是比较简单粗暴的，更合理的方式当然是按需加载，特别是大型项目中尤为重要。现在有很多现成的工具支持lazyload，requirejs原生就支持，当然跟AngularJS也有很好的集成工具，在模块层面，controller层面都能做。这里涉及到更重要的点其实是合理的模块差分，公共模块的分离，复用范围的控制，更合理的划分加载模式才能达到更好的加载性能。AngularJS内置组件的定义很规范，也正是这种关注点分离的思想保证了前端模块的可测性。

### 模板

模板就是HTML字符串，在元素或者属性中定义的AngularJS表达式会包含在模板里，AngularJS和其他框架最大的一个不同点就在于AngularJS的模板不是一种需要被转化为HTML的中间格式（mustache.js和handlerbars就是这种套路）。
初始时，每个单页应用加载 `index.html` 文件。在AngularJS的应用中，这个文件会包含一组标准和自定义的HTML属性，元素和注释，这些东西会配置并且拉起应用。每个后续的用户动作只需要加载另外的模板或者改变应用的状态，例如通过框架提供的数据绑定。

**简单模板**

```HTML
<html ng-app>
 <!-- Body tag augmented with ngController directive  -->
 <body ng-controller="MyController">
   <input ng-model="foo" value="bar">
   <!-- Button tag with ng-click directive, and
          string expression 'buttonText'
          wrapped in "{{ }}" markup -->
   <button ng-click="changeFoo()">{{buttonText}}</button>
   <script src="angular.js"></script>
 </body>
</html>
```

模板通过AngularJS表达式定义对于什么样的用户操作会触发什么样的动作被执行。在上面的例子中， `ng-click` 属性表明在当前作用域下的 `changeFoo` 方法会在用户点击之后执行。

**注解：** AngularJS的模板支持很多声明式语法。为了语义，表现和结构的分离，前端一直推荐非侵入式的JS编程。AnguarJS反其道而行之，支持很多与页面绑定的行为定义，个人感觉完美主义还是要结合现实的，毕竟侵入式的写法真的很方便，而且其实很多行为本身就是是和语义结构绑定的。结合UI-Router可以实现更方便的自状态页面控制，让模板的渲染更加灵活。

### 控制器

AngularJS的控制器就是Javascript的方法，通过在 *scope* 上面绑定方法来处理用户和应用的交互（例如鼠标事件，键盘事件等等）。对于控制器需要的外部依赖组件都通过AngularJS的依赖注入机制提供。控制器也通过在 *scope* 上面绑定数据提供模板的 *model*。我们能把这种数据当作 *view model*。

```JavaScript
function MyController($scope) {
  $scope.buttonText = 'Click me to change foo!';
  $scope.foo = 42;

  $scope.changeFoo = function () {
    $scope.foo += 1;
    alert('Foo changed');
  };
}
```

举个例子，如果我们把上面的样例控制器绑在前面章节示例的模板上面，用户就能和应用进行一些不同方式的交互。

1. 通过在文本输入框中输入文字改变 `foo` 的值。因为双向绑定机制，输入的值将会立即反映到 `foo` 的值上面。
2. 通过点击带有 `Click me to change foo!` 标签的按钮也能改变 `foo` 的值

如果之前被定义过，所有的自定义元素，属性，注释或者类都能被识别为AngularJS *指令*。

**注解：** 控制器大都是跟页面绑定的，可复用性相对较差，跟之前用Jquery做页面的思路其实大致相同，都是做一些初始化，数据缓存，事件绑定。就是通过AJAX局部刷新的时候比较方便，双向绑定的机制让我们只需要关心维护视图模型，视图模型变动引起的视图改变通过监听视图模型的指令来完成，很多不复杂并且相同的视图更新操作都能封装起来。MVC在前端也有很多尝试，但个人觉得Controller一层可能有点鸡肋，一般场景下的视图模型和视图的关联都是很固定的，这也是现在MVVM更流行的其中一个原因吧。忍不住赞一下依赖注入特性，真的是开箱即用，只是为了压缩写法上稍微有些不自然，不过现在好像有工具能支持去掉二次声明了。

### 作用域

AngularJS的作用域就是个暴露给模板的Javascript对象。作用域能够包含不同的属性 -内置类型，对象和方法。所有绑定在作用域上面的方法能够通过AngularJS指定作用域的模板里面定义的表达式求值被调用，也能够保留作用域的引用被组件直接调用。通过使用合适的 *指令*，关联到作用域的数据能够被绑定到视图上。每次页面的变动都会反映到作用域属性上，同时每次作用域属性的变动都会反映到页面上。

作用域通过原型链连接（除了以 *isolated* 显示开头的作用域）是任何AngularJS应用的另外一个重要作用域特性。因为子作用域的父级属性是它们的直接或者间接原型，这样使得任何的子作用域就能调用父作用域的方法。

作用域继承通过下面的例子说明：

```HTML
<div ng-controller="BaseCtrl">
  <div id="child" ng-controller="ChildCtrl">
    <button id="parent-method" ng-click="foo()">Parent method</button>
    <button ng-click="bar()">Child method</button>
  </div>
</div>
```

```JavaScript
function BaseCtrl($scope) {
  $scope.foo = function () {
    alert('Base foo');
  };
}

function ChildCtrl($scope) {
  $scope.bar = function () {
    alert('Child bar');
  };
}
```

`div#child` 绑上 `ChildCtrl` ，但是因为注入到 `ChildCtrl` 中的作用域原型继承自父作用域 (例如 注入到 `BaseCtrl` 中的作用域) `foo` 方法能够被 `button#parent-method` 访问到。

**注解：** Javascript本身没有继承机制，所有的所谓继承都是通过原型链实现的，AngularJS的作用域也就有了原型链的特性，当前没有向上找，一般情况下不推荐这种依赖继承层级关系的数据传递，父子级别的数据依赖还是通过pub/sub机制比较好，需求变动改动起来也方便。有时候依赖这种层次关系的数据产生的bug调起来真的很头大，通过Controller As的语法能够更好避免类似的问题。

### 指令

所有DOM操作应该被放在指令里面。作为一条经验法则，当你在Controller里面有DOM操作的需要，要么你就弄个新的出来，要么重构已经有的能够处理必要DOM操作的指令。
每个指令都会有个名字和相关联的逻辑。最简单的指令示例中仅仅包含了名字和 *postLink* 方法的定义，所有必要的逻辑都被封装在指令里面。更复杂的指令中可能包含一些属性，例如：

- template
- compile 方法
- link 方法
- 等等...

通过引用指令的名字，指令能够在声明式模板中被使用。

例子:

```JavaScript
myModule.directive('alertButton', function () {
  return {
    template: '<button ng-transclude></button>',
    scope: {
      content: '@'
    },
    replace: true,
    restrict: 'E',
    transclude: true,
    link: function (scope, el) {
      el.click(function () {
        alert(scope.content);
      });
    }
  };
});
```

```HTML
<alert-button content="42">Click me</alert-button>
```

上面例子中的标签 `<alert-button></alert-button>` 会被按钮元素给替换。当用户点击按钮，字符串 `42` 会被弹出来。

因为这篇文章不是要解释AngularJS的复杂API，我们对于指令的描述在这里打住。

**注解：** 指令作为AngularJS对于Web Components的实现，在一定程度上实现了UI的组件化开发，但是基于视图模型的限制，AngularJS能够实现的组件功能相对较弱，因为双向绑定的特性也带来了AngularJS组件的性能瓶颈。有时候一个指令的外部接口输入输出格式是什么样的，哪些要双向绑定，哪些要单向绑定，哪些要有回调方法可能很难一次定义清楚，要不断重构和积累。其实ReactJS在UI组件化上的效果更好，因为只关注视图层，它可以很方便的继承到现有系统中，也有像Angular-react这样的项目可以结合两者的优势。

### 过滤器

AngularJS的过滤器就是用来封装格式化数据的必要逻辑。一般过滤器用在模板里面，但是在控制器，指令， *服务* 和其他的过滤器里面通过依赖注入也是能够访问到的。

这里有个样例过滤器的定义，就是把指定的字符串中字母转化为大写。

```JavaScript
myModule.filter('uppercase', function () {
  return function (str) {
    return (str || '').toUpperCase();
  };
});
```

在模板里面这个过滤器能够通过Unix的管道语法来使用：

```HTML
<div>{{ name | uppercase }}</div>
```

在控制器里面，过滤器能够像下面一样使用：

```JavaScript
function MyCtrl(uppercaseFilter) {
  $scope.name = uppercaseFilter('foo'); //FOO
}
```

**注解：** 大部分情况下，AngularJS的内置过滤器就能满足我们大部分的应用需要了，结合业务需要会有少量的补充，本质上就是对于数据集的重新格式化，结合列表可以实现排序或者预处理的工作，用惯了Linux系统看着这语法还是很亲切的。

### 服务

任何一段不属于上面描述组件的逻辑块应该被放在服务中。通常服务封装领域相关逻辑，持久化逻辑，异步请求，长连接等等。当应用中的控制器变得越来越臃肿，重复的代码应该放到服务中。

```JavaScript
myModule.service('Developer', function () {
  this.name = 'Foo';
  this.motherLanguage = 'JavaScript';
  this.live = function () {
    while (true) {
      this.code();
    }
  };
});
```

服务能够被注入到任何支持依赖注入的组件中（控制器，其他服务，过滤器，指令）。

```JavaScript
function MyCtrl(developer) {
  var developer = new Developer();
  developer.live();
}
```

**注解：** 服务可以看作我们应用开发中的Util方法集，通过特定功能和应用场景分组，将重复的代码抽取出来，因为需求上一般呈现出的是功能上的一致性，公共需求的改动只需要在单一的地方处理就好了。例如，异步请求的调用，当我们需要统一处理后台异常状态码的时候，改动可以最小化，同时不会对原有业务逻辑产生影响。有些公共服务是能够直接识别出来的，更多时候在业务需求基本稳定之后，Copy&Paste的代码就要通过重构提取成一个服务。

## AngularJS 模式

下面的章节，我们就要看看传统的设计和架构模式是怎样在AngularJS的组件中组合应用的。

最后一章我们会看看一些经常在AngularJS单页应用开发（不仅限于此）中使用的架构模式。

### 服务

#### 单例

>单例模式是一种限制类的实例只有一个的设计模式。 当只需要一个对象来协调系统中的操作时，这种设计模式很有用。这种概念有时指代只有一个对象存在会更高效运作或者将初始化限制在一定数量的对象上的系统。

下面的UML图中描绘了单例设计模式。

![单例](https://rawgit.com/mgechev/angularjs-in-patterns/master/images/singleton.svg "Fig. 1")

当指定了任何组件需要的依赖之后，AngularJS是遵循下面的算法处理的：

- 取得依赖组件的名字并且在定义在词法闭包中的哈希表里查找（所以它具有私有可见性）。
- 如果依赖存在，AngularJS将它作为参数传递给需要它的组件。
- 如果依赖不存在:
  - AngularJS 通过调用依赖提供者的工厂方法将它实例化（例如 `$get`）。为了处理所有的依赖需要的必要依赖，初始化依赖可能需要基于相同的算法递归的调用。这个过程可能会造成循环以来。
  - AngularJS 在上面提到的哈希表里面缓存它。
  - AngularJS 将它作为参数传递给依赖它的组件。

我们能够在AngularJS源码中的 `getService` 方法里看到清楚的实现：

```JavaScript
function getService(serviceName) {
  if (cache.hasOwnProperty(serviceName)) {
    if (cache[serviceName] === INSTANTIATING) {
      throw $injectorMinErr('cdep', 'Circular dependency found: {0}', path.join(' <- '));
    }
    return cache[serviceName];
  } else {
    try {
      path.unshift(serviceName);
      cache[serviceName] = INSTANTIATING;
      return cache[serviceName] = factory(serviceName);
    } catch (err) {
      if (cache[serviceName] === INSTANTIATING) {
        delete cache[serviceName];
      }
      throw err;
    } finally {
      path.shift();
    }
  }
}
```

我们能够把所有的服务当作是单例的，因为所有的服务都不会被实例化两次。我们能够把缓存看作单例管理器。这里跟标准的单例有点不一样，我们不是在构造方法中保存对于单例的静态私有引用，而是在单例管理器（在上面的代码片段中定义为 `cache`）里面直接保留了引用。

通过这种方式服务实际上都是单例但是不是通过单例模式实现的，这样比标准的实现提供了一些便利：

- 它提升了你源代码的可测性
- 你能控制单例对象的创建（在我们的例子中IoC容器通过延迟实例化单例为我们控制它）

对相关主题的讨论可以看 Misko Hevery 在谷歌测试博客上的 [文章](http://googletesting.blogspot.com/2008/05/tott-using-dependancy-injection-to.html) 。

#### 工厂方法

>工厂方法模式是一种创建者模式，工厂方法用来处理对象的创建而不需要指定具体将要被创建对象所对应的类。 这是通过一个工厂方法来创建对象来做到的，可能是在一个实现定义接口（抽象类） 的类（实现类）或者在基类中实现，通过继承类覆盖，而不是通过构造器。

![工厂方法](https://rawgit.com/mgechev/angularjs-in-patterns/master/images/factory-method.svg "Fig. 2")

让我们看下面的代码片段:

```JavaScript
myModule.config(function ($provide) {
  $provide.provider('foo', function () {
    var baz = 42;
    return {
      //Factory method
      $get: function (bar) {
        var baz = bar.baz();
        return {
          baz: baz
        };
      }
    };
  });
});

```

上面的代码我们使用 `config` 回调来定义新的 "provider". Provider 是个新对象, 有个叫 `$get`的方法。因为在JavaScript里我们不会有接口，并且这种语言是鸭子类型的，对于providers有这么个命名工厂方法的约定。

每个服务，过滤器，指令和控制器都有一个provider (例如 对象有个叫 `$get` 的工厂方法), 负责创建组件的实例。

我们可以深入挖掘下AngularJS的实现

```JavaScript
//...

createInternalInjector(instanceCache, function(servicename) {
  var provider = providerInjector.get(servicename + providerSuffix);
  return instanceInjector.invoke(provider.$get, provider, undefined, servicename);
}, strictDi));

//...

function invoke(fn, self, locals, serviceName){
  if (typeof locals === 'string') {
    serviceName = locals;
    locals = null;
  }

  var args = [],
      $inject = annotate(fn, strictDi, serviceName),
      length, i,
      key;

  for(i = 0, length = $inject.length; i < length; i++) {
    key = $inject[i];
    if (typeof key !== 'string') {
      throw $injectorMinErr('itkn',
              'Incorrect injection token! Expected service name as string, got {0}', key);
    }
    args.push(
      locals && locals.hasOwnProperty(key)
      ? locals[key]
      : getService(key)
    );
  }
  if (!fn.$inject) {
    // this means that we must be an array.
    fn = fn[length];
  }

  return fn.apply(self, args);
}
```

从上面的例子中我们能够看到 `$get` 方法实际是如何被使用的：

```JavaScript
instanceInjector.invoke(provider.$get, provider, undefined, servicename)
```

上面的代码片段通过调用 `instanceInjector` 的 `invoke` 方法将指定服务的工厂方法（例如 `$get`）作为第一个参数来调用。 在 `invoke` 方法内工厂方法作为第一个参数被 `annotate` 调用。注解通过AngularJS的依赖注入机制处理所有的依赖，就像上面考虑到的。当所有的依赖都被处理好之后工厂方法被调用： `fn.apply(self, args)`。

如果我们按照UML图来考虑上面的行为，我们能够称这个provider是个 "ConcreteCreator"，而实际的组件是生产出来的"Product"。

在这种场景下因为不是直接创建对象，使用工厂模式有一些好处。通过这种方式框架能够在新组件实例化的时候处理好样板：

- 组件需要被实例化的最佳时机
- 处理好组件必要的所有依赖
- 指定组件被允许创建的最多实例数量（对于服务和过滤器只有单个，但是对于控制器会有多个）

#### 装饰器

>装饰器模式（配器模式中也被称作包装器）是允许在不影响类创建的其他对象前提下将行为静态地或者动态地添加到独立对象上的一种设计模式。

![装饰器](https://rawgit.com/mgechev/angularjs-in-patterns/master/images/decorator.svg "Fig. 4")

AngularJS 提供了开箱即用扩展或者增强已有服务功能的方式。通过使用 `$provide` 的 `decorator` 方法，你可以创建任何先前定义服务或者第三方服务的包装器：

```JavaScript
myModule.controller('MainCtrl', function (foo) {
  foo.bar();
});

myModule.factory('foo', function () {
  return {
    bar: function () {
      console.log('I\'m bar');
    },
    baz: function () {
      console.log('I\'m baz');
    }
  };
});

myModule.config(function ($provide) {
  $provide.decorator('foo', function ($delegate) {
    var barBackup = $delegate.bar;
    $delegate.bar = function () {
      console.log('Decorated');
      barBackup.apply($delegate, arguments);
    };
    return $delegate;
  });
});
```

上面的例子定义了叫做 `foo` 的新服务。在 `config` 的回调中我们想要包装服务的名称 `"foo"`作为第一个参数被 `$provide.decorator` 调用，第二个工厂方法参数实现了真正的包装。 `$delegate` 保存了原始服务 `foo` 的引用。依赖于AngularJS的依赖注入机制，对于局部依赖的引用被作为构造函数的第一个参数传进去。
我们通过覆盖 `bar` 方法来装饰服务。实际的装饰只是通过额外调用 `console.log statement` - `console.log('Decorated');` 并且在这之后调用原始的有适当上下文的 `bar` 方法来扩展服务的 `bar` 方法。

这种模式尤其在我们需要修改第三方服务的时候有用。如果需要多个相似的装饰器需要被添加（像对于多个方法的性能评估，授权，日志等等），我们会有很多的重复并且违背DRY原则. 在这种情况下使用[面向切面编程](http://en.wikipedia.org/wiki/Aspect-oriented_programming)会更好. 我所知道AngularJS最好的AOP框架是这个 [github.com/mgechev/angular-aop](https://github.com/mgechev/angular-aop).

#### 门面

>门面就是个封装了一大坨代码（例如某个类库），仅仅提供最简单接口的对象。门面能够：

>1. 让软件库更好用，理解并且测试，因为门面对于公共的任务有便捷的方法；

>2. 基于同样的原因让软件库可读性更好；

>3. 在库内部减少对于外部代码的依赖，因为大部分代码使用门面，这样允许在系统中有更多的复杂性；

>4. 将定义的很烂的一组API用一个设计良好的API来封装（按照不同任务的需要）。

![门面](https://rawgit.com/mgechev/angularjs-in-patterns/master/images/facade.svg "Fig. 11")

在AngularJS中有很多门面. 每次要为给定功能提供更高层次API的时候，你实际上就是在创建一个门面。

例如，让我们看看我们怎么创建一个 `XMLHttpRequest` POST 请求：

```JavaScript
var http = new XMLHttpRequest(),
    url = '/example/new',
    params = encodeURIComponent(data);
http.open("POST", url, true);

http.setRequestHeader("Content-type", "application/x-www-form-urlencoded");
http.setRequestHeader("Content-length", params.length);
http.setRequestHeader("Connection", "close");

http.onreadystatechange = function () {
  if(http.readyState == 4 && http.status == 200) {
    alert(http.responseText);
  }
}
http.send(params);
```
但是如果你想要将数据通过AngularJS的 `$http` 服务来发送，你会这样写:

```JavaScript
$http({
  method: 'POST',
  url: '/example/new',
  data: data
})
.then(function (response) {
  alert(response);
});
```
or we can even:

```JavaScript
$http.post('/someUrl', data)
.then(function (response) {
  alert(response);
});
```
第二种选择提供了预配置的版本，对于指定的URL创建了一个HTTP的POS请求。

更高程度的抽象通过 `$resource` 创建,  `$resource` 是基于 `$http` 服务的. 我们会在[活动记录](#active-record)和[代理](#proxy)章节中进一步讨论 `$http` 服务.

#### 代理

>一个代理，一般化的形式上，是某个类作为其他东西的接口工作。代理能够作为一切的接口： 网络连接，内存中的大对象，文件或者一些其他难以或者不能复制的资源。

![代理](https://rawgit.com/mgechev/angularjs-in-patterns/master/images/proxy.svg "Fig. 9")

我们可以分出三种不同类型的代理：

- 虚拟代理
- 远程代理
- 受保护代理

在这个子章节中我们会看看AngularJS对于虚拟代理的实现。

在下面的代码片段中，调用了叫 `User` 的 `$resource` 实例方法 `get` ：

```JavaScript
var User = $resource('/users/:id'),
    user = User.get({ id: 42 });
console.log(user); //{}
```

`console.log` 会打印出来个空对象。因为AJAX请求发生在后台，当 `User.get` 被调用的时候是异步的，在 `console.log` 被调用的时候，我们还没有获得实际的user。在 `User.get` 发送GET请求之后，仅仅返回一个空对象并且保留对于它的引用。我们可以把这个对象当作虚拟代理（一个简单的占位符）。在客户端获取到服务器上实际的数据之后这个占位符才会被填充。

AngularJS是怎么做到的呢? 好了，让我们看看下面的代码片段：

```JavaScript
function MainCtrl($scope, $resource) {
  var User = $resource('/users/:id'),
  $scope.user = User.get({ id: 42 });
}
```

```html
<span ng-bind="user.name"></span>
```
开始的时候，上面的代码片段执行， `$scope` 对象的 `user` 属性会被赋予一个空对象 (`{}`)，这意味着 `user.name` 将会是未定义，啥也不会渲染。AngularJS内部会保留这个空对象的引用。一旦服务器返回get请求的数据，AngularJS 会用从服务其上获取到的数据填充对象。在下个 `$digest` 循环中AngularJS会检测到 `$scope.user`的变化, 进而触发视图的渲染。

#### 活动记录

>活动记录是一种包含了数据和行为的对象。通常这种对象的大部分数据是持久化的。活动记录的职责就是处理好和数据库增删改查的通信。它可能会将这种职责代理给更低级别的对象，但是会调用活动记录的实例或者静态方法来和数据库通信。

![活动记录](https://rawgit.com/mgechev/angularjs-in-patterns/master/images/active-record.svg "Fig. 7")

AngularJS 定义了叫 `$resource` 的服务。在当前的AngularJS版本(1.2+)，它是排除在AngularJS的核心模块之外的.

依据AngularJS的文档， `$resource` 是：

>一个创建了跟Restful服务器数据源进行交互的资源对象工厂实例
>资源对象返回的对象有更高层次的方法，这样就不需要直接使用相对底层的$http服务

`$resource` 是这样用的:

```JavaScript
var User = $resource('/users/:id'),
    user = new User({
      name: 'foo',
      age : 42
    });

user.$save();
```

调用 `$resource` 会为我们的模型实例创建一个构造方法。每个模型示例都有用于CRUD的方法。

这样我们就能够像下面这样使用构造方法和上面的静态方法:

```JavaScript
User.get({ userid: userid });
```

上面的代码会立刻返回个空对象并且保留它的引用。一旦成功返回并且被解析，AngularJS 会用回去到的数据填充这个对象 (看 [代理](#proxy))。

你可以找到更多关于 `$resource` [$resource的魔法](http://blog.mgechev.com/2014/02/05/angularjs-resource-active-record-http/) and [AngularJS 的文档](https://docs.angularjs.org/api/ngResource/service/$resource) 的信息。

鉴于 Martin Fowler 这样说

> 活动记录的责任就是为了CRUD跟数据库打交道

`$resource` 没有严格意义上实现活动记录模式，因为它是跟Restful服务而不是数据库打交道。不管怎么样，我们可以把它当作“像活动记录一样的Restful通信”。

#### 拦截过滤器

>在web请求中，为实现公共的预处理和后处理创建一个支持组合的过滤器链。

![Composite](https://rawgit.com/mgechev/angularjs-in-patterns/master/images/intercepting-filters.svg "Fig. 3")

有时候你需要对HTTP请求做一些预处理或者后处理。你在HTTP的请求/响应中做预处理或者后处理来包含日志，安全或者其他会被请求头部和消息体影响的东西。基本上拦截过滤器模式包含一个过滤器链，在链上的过滤器都会按照指定的顺序处理数据。每个过滤器的输出都是下个的输入。

在AngularJS里面 `$httpProvider` 用了拦截过滤器的模式。 `$httpProvider` 有个叫 `interceptors` 的数组, 包含了一堆对象。每个对象都有叫: `request`, `response`, `requestError`, `responseError` 的属性。

`requestError` 是个拦截器, 在前面的拦截器抛出错误或者在promise中被拒绝掉的时候会被调用，同样的，`responseError` 会在前一个 `response` 拦截器抛出错误之后被调用。

这有个使用对象语法创建拦截器的基本例子：

```JavaScript
$httpProvider.interceptors.push(function($q, dependency1, dependency2) {
  return {
   'request': function(config) {
       // same as above
    },
    'response': function(response) {
       // same as above
    }
  };
});
```

### 指令

#### 组合

>组合模式是个隔离设计模式。组合模式是说，一组对象按照同样的方式被当作对象的某个实例属性。组合的目的是将对象组合到一个树形结构中来代表局部和整体的层次关系。

![组合](https://rawgit.com/mgechev/angularjs-in-patterns/master/images/composite.svg "Fig. 3")

基于经典设计模式理论，MVC就是下面模式的组合：

- 策略
- 组合
- 观察者

理论中定义视图是组件的组合。在AngularJS中有相似的解决方案。我们的视图由指令和DOM元素的组合来构成，指令解释放在视图上的。

让我们看看下面的例子：

```HTML
<!doctype html>
<html>
  <head>
  </head>
  <body>
    <zippy title="Zippy">
      Zippy!
    </zippy>
  </body>
</html>
```

```JavaScript
myModule.directive('zippy', function () {
  return {
    restrict: 'E',
    template: '<div><div class="header"></div><div class="content" ng-transclude></div></div>',
    link: function (scope, el) {
      el.find('.header').click(function () {
        el.find('.content').toggle();
      });
    }
  }
});
```

这个例子定义了一个简单的UI组件指令。这个叫zippy的组件有头部和内容。点击头部会触发内容的显示或者隐藏。

从第一个例子里面我们能注意到整个DOM树是元素的组合。根元素是 `html` 元素，直接跟着嵌套的 `head` 和 `body` 元素等等...

在第二个Javascript的例子中，我们看到指令的 `template` 里面包含 `ng-transclude` 标记。这意味着在指令 `zippy` 里面我们有另外一个叫 `ng-transclude` 的指令, 也就是指令的组合。理论上我们能够在到达叶子节点之前一直嵌套组件。

### 解释器

>在软件编程中，解释器模式是一种指定如何解析语言中句子的设计模式。最基本的思想就是给每个符号（终止符或者非终止符）作为某种语言中的类。语言里句子的语法树就是组合模式中的一个实例并且被用来解析句子。

![解释器](https://rawgit.com/mgechev/angularjs-in-patterns/master/images/interpreter.svg "Fig. 6")

在 `$parse` 服务里AngularJS提供了自己对于领域驱动语言解析器的实现。被使用的DSL是被简化和修改过的Javascript语言。
Javascript表达式和AngularJS表达式最大的不同在于AngularJS表达式：

- 可能包含像UNIX管道语法的过滤器
- 不会抛出任何错误
- 没有任何流程控制语句（异常，循环，if声明尽管你能使用三元运算符）
- 在指定上下文中被解析（当前 `$scope` 的上下文）

在 `$parse` 服务中有两个主要的组件：

```JavaScript
//Responsible for converting given string into tokens
var Lexer;
//Responsible for parsing the tokens and evaluating the expression
var Parser;
```

一旦指定表达式被标记化，基于性能的考虑就会被内部缓存。

AngularJS DSL里面终止性的表达式是这样定义的：

```JavaScript
var OPERATORS = {
  /* jshint bitwise : false */
  'null':function(){return null;},
  'true':function(){return true;},
  'false':function(){return false;},
  undefined:noop,
  '+':function(self, locals, a,b){
        //...
      },
  '*':function(self, locals, a,b){return a(self, locals)*b(self, locals);},
  '/':function(self, locals, a,b){return a(self, locals)/b(self, locals);},
  '%':function(self, locals, a,b){return a(self, locals)%b(self, locals);},
  '^':function(self, locals, a,b){return a(self, locals)^b(self, locals);},
  '=':noop,
  '===':function(self, locals, a, b){return a(self, locals)===b(self, locals);},
  '!==':function(self, locals, a, b){return a(self, locals)!==b(self, locals);},
  '==':function(self, locals, a,b){return a(self, locals)==b(self, locals);},
  '!=':function(self, locals, a,b){return a(self, locals)!=b(self, locals);},
  '<':function(self, locals, a,b){return a(self, locals)<b(self, locals);},
  '>':function(self, locals, a,b){return a(self, locals)>b(self, locals);},
  '<=':function(self, locals, a,b){return a(self, locals)<=b(self, locals);},
  '>=':function(self, locals, a,b){return a(self, locals)>=b(self, locals);},
  '&&':function(self, locals, a,b){return a(self, locals)&&b(self, locals);},
  '||':function(self, locals, a,b){return a(self, locals)||b(self, locals);},
  '&':function(self, locals, a,b){return a(self, locals)&b(self, locals);},
  '|':function(self, locals, a,b){return b(self, locals)(self, locals, a(self, locals));},
  '!':function(self, locals, a){return !a(self, locals);}
};
```

我们可以认为每个跟终止性表达符关联的方法是对 `AbstractExpression` 接口的实现。

每个 `Client` 在指定上下文中解析特定的AngularJS表达式 - 特定的作用域.

AngularJS表达式样例如下：

```JavaScript
// toUpperCase filter is applied to the result of the expression
// (foo) ? bar : baz
(foo) ? bar : baz | toUpperCase
```

#### 模板视图

> 在页面上通过嵌入的标记语法渲染信息。

![模板视图](https://rawgit.com/mgechev/angularjs-in-patterns/master/images/template-view.svg "Fig. 8")

动态页面的渲染不是那么简单的事情。这里面掺和着一大堆的字符串连接，操作，还伴随着挫败感。写你自己的标记语言并且嵌入少量的表达式是创建动态页面更简单的方式，标记语言后续会被在特定的上下文里被解析出来，整个模板最终会被编译成最终的格式，在我们的情况下将会是HTML(或者可能是DOM)。这就是模板引擎做的事情 - 它们使用指定的DSL，在合适的上下文里解析并且将它转为最终的格式。

模板更广泛用于后端。例如，你可以在HTML里面嵌入PHP代码来创建动态页面，你可以使用Smarty或者你可以为了在静态页面中嵌入Ruby代码使用eRuby。

对于Javascript有很多的模板引擎，例如mustache.js, handlerbars等等。大部分引擎将模板作为字符串来操作。模板可以放在不同的地方 - 通过AJAX作为静态文件加载，作为 `script` 嵌入到你的视图中甚至是在Javascript里面。

例如：

```html
<script type="template/mustache">
  <h2>Names</h2>
  {{#names}}
    <strong>{{name}}</strong>
  {{/names}}
</script>
```

模板引擎通过在特定上下文里面编译它，将这个字符串转化成DOM元素。所有内嵌在标记语言中的表达式被解析并且替换为实际的值。

例如如果你要在下面对象 `{ names: ['foo', 'bar', 'baz'] }` 的上下文里面解析上面的模板，我们就会得到：

```html
<h2>Names</h2>
  <strong>foo</strong>
  <strong>bar</strong>
  <strong>baz</strong>
```

AngularJS模板实际就是HTML，而不是如同传统模板一样仅仅是个中间的格式。
AngularJS编译器就是在DOM树上遍历并且找到已知的指令（元素，属性，类甚至是注释）。当AngularJS找到任何这些指令的时候，它会调用相关的逻辑，这也会包含在当前作用域下对于其他表达式的解析。

例如:

```html
<ul ng-repeat="name in names">
  <li>{{name}}</li>
</ul>
```

在作用域的上下文：

```javascript
$scope.names = ['foo', 'bar', 'baz'];
```

会产生和上面一样的结果。这里主要的不同点在于模板不是包含在 `script` 标签而是HTMl里面。


### 作用域

#### 观察者

>观察者模式是一种这样的软件设计模式：有个称作主题的对象，维护了它所依赖观察者的列表，在任何状态改变的时候，自动通知所有的观察者，通常就是调用观察者的某个方法。这种模式主要用于实现分布式的时间处理系统。

![观察者](https://rawgit.com/mgechev/angularjs-in-patterns/master/images/observer.svg "Fig. 7")

在AngularJS应用中有两种在作用域之间通信的基本方式。第一种方式通过在子作用域中调用父作用域中的方法。能这样做是因为子作用域通过原型继承自父作用域，上面有提到（看[作用域](#scope)）。这样允许我们进行单向的通信 - 子作用域到父作用域。有时候这在特定作用域中调用方法是很必要的。AngularJS提供了内置的观察者模式，允许这样做。另一个可能的观察者模式的用例是，当多个作用域对于特定的事件感兴趣的时候，触发事件的作用域并不会察觉到它。这样使得在不同作用域之间解耦成为可能，任何作用域都不应该察觉到其他作用域的存在。

每个AngularJS的作用域都有共有的方法 `$on`, `$emit` 和 `$broadcast`。方法 `$on` 接受主题作为第一个参数并且将回调作为第二个。我们可以把回调当成是个观察者 - 一个实现了 `Observer` 接口的对象（在Javascript中函数是一等公民，所以你只需要实现 `notify` 方法）：

```JavaScript
function ExampleCtrl($scope) {
  $scope.$on('event-name', function handler() {
    //body
  });
}
```

通过这种方式，当前的作用域“订阅”类型为 `event-name` 的事件. 当 `event-name` 事件在任何父或者子作用域中被触发的时候， `handler` 就会被调用。

方法 `$emit` 和 `$broadcast` 在作用域链上分别用来触发向上传递或者向下传递的事件。
例如：

```JavaScript
function ExampleCtrl($scope) {
  $scope.$emit('event-name', { foo: 'bar' });
}
```

上面例子中的作用域，向上级所有作用域触发 `event-name` 事件。这意味着所有父作用域上监听了 `event-name` 事件的作用域会被通知并且上面的回调方法会被调用。

当 `$broadcast` 方法被调用的时候也是相似的。唯一不同的地方在于事件是被向下传递的 - 传给所有的子作用域。
每个作用域能够订阅任何事件，并且指定多个回调 （也就是说可以对于指定的事件关联多个观察者）。

在JavaScript社区这一模式作为pus/sub更为人所知。

#### 责任链

>责任链模式是由一堆命令对象和一系列处理对象组成的一种设计模式。每个处理对象包含定义它能够处理的命令对象类型逻辑，它不能处理的命令类型被传递给责任链上下一个处理对象。在责任链尾部添加新的处理对象也是被支持的一种机制。

![责任链](https://rawgit.com/mgechev/angularjs-in-patterns/master/images/chain-of-responsibilities.svg "Fig. 5")

像上面说到的，AngularJS应用中的作用域构建了一个层次结构的作用域链。有些作用域是“独立的”，这意味着它们不会原型继承它们的父作用域，而是通过它们的 `$parent` 属性连接。

当 `$emit` 或者 `$broadcast` 被调用的时候，我们可以把作用域链当作事件巴士，或者更准确地说是责任链。一旦事件被触发，它就会被向上或者向下传递（依赖被调用的方法）。每个子序列的作用域会：

- 处理事件并且继续向下一级作用域传递
- 处理事件，阻止传递
- 直接向下一级作用域传递而不做任何处理
- 阻止传递并且不做任何处理

在下面的例子里面你会看到 `ChildCtrl` 触发了一个时间，该事件在作用域链上被向下传递。在上面的例子里面每个父作用域（在 `ParentCtrl` 中用到的和在 `MainCtrl` 用到的）会通过在控制台里打印 `"foo received"` 来处理事件。如果任何作用域需要被当作最后的处理目标，它可以调用通过回调传递获得到的事件对象上的 `stopPropagation` 方法来实现。

```JavaScript
myModule.controller('MainCtrl', function ($scope) {
  $scope.$on('foo', function () {
    console.log('foo received');
  });
});

myModule.controller('ParentCtrl', function ($scope) {
  $scope.$on('foo', function (e) {
    console.log('foo received');
  });
});

myModule.controller('ChildCtrl', function ($scope) {
  $scope.$emit('foo');
});
```

在上面UML图中的不同处理器就是不同被注入到控制器中的作用域。

#### 命令

>在面向对象编程中，命令模式是一种行为设计模式。说的是使用一个对象来表征和包裹之后需要调用的某个方法所必要的信息。这些信息包括方法名，包含方法和方法调用参数值的对象。

![命令](https://rawgit.com/mgechev/angularjs-in-patterns/master/images/command.svg "Fig. 11")

在继续说命令模式的应用之前让我们讲讲AngularJS是如何实现数据绑定的。

当我们要把模型绑定到视图上的时候，我们使用指令 `ng-bind` （为了单向的数据绑定）和 `ng-model` （为了双向的数据绑定）。例如我们想要模型 `foo` 的每一个改动都放映到视图上，我们可以：

```html
<span ng-bind="foo"></span>
```

现在每次我们改动 `foo` 的值span内部的文本都会被改变。我们能够能够通过更加复杂的AngularJS表达式来获得同样的效果：

```html
<span ng-bind="foo + ' ' + bar | uppercase"></span>
```

在上面的示例中，span的值将会是 `foo` 和 `bar` 连接起来并且首字母大写的值。究竟发生了什么呢？

每个 `$scope` 都有个叫 `$watch` 的方法。当AngularJS编译器找到 `ng-bind` 指令，它会创建一个新的对于 `foo + ' ' + bar | uppercase` 表达式的监听器也就是 `$scope.$watch("foo + ' ' + bar | uppercase", function () { /* body */ });`。回调会在每次表达式的值改变的时候被触发。在这个例子中回调方法会更新span的值。

下面是 `$watch` 方法实现的头几行：

```javascript
$watch: function(watchExp, listener, objectEquality) {
  var scope = this,
      get = compileToFn(watchExp, 'watch'),
      array = scope.$$watchers,
      watcher = {
        fn: listener,
        last: initWatchVal,
        get: get,
        exp: watchExp,
        eq: !!objectEquality
      };
//...
```

我们可以把 `watcher` 对象当作一条命令。在每次[`"$digest"`](https://docs.angularjs.org/api/ng/type/$rootScope.Scope#$digest)循环的时候命令表达式都会被解析。一旦AngularJS检测到表达式里面的改动，它调用 `listener` 方法。 `watcher` 命令封装了对于特定表达式的监听并且将命令的执行代理到 `listener` （实际的接受者）。我们可以把 `$scope` 当作命令的 `客户端` ，把 `$digest` 循环当作命令的 `调用者`.

### 控制器

#### 页面控制器

>一个处理网站上特定页面或者动作请求的对象。 Martin Fowler

![页面控制器](https://rawgit.com/mgechev/angularjs-in-patterns/master/images/page-controller.svg "Fig. 8")

根据 [4](#references) 页面控制器是：

>页面控制器模式接受页面请求的输入，调用模型上被请求动作，并且决定正确的渲染页面。将分发的逻辑从视图相关的代码中分离出来

因为在不同的页面上有许多重复的行为（像渲染页头，页脚，处理用户的回话等等）页面控制器能够构成层次结构。在AngularJS里我们使用的控制器担负的责任很有限。它们不接受用户请求，因为这时 `$route` 或者 `$state` 服务的责任，页面渲染的责任归 `ng-view`/`ui-view` 所有。

跟页面控制器相似的是，AngularJS控制器处理用户的交互，提供和更新模型。当模型绑定到作用域上，它就暴露给了视图，所有由于用户动作被视图调用的方法都是已经绑定在作用域上的。另一个页面控制器和AngularJS控制器的相似点就是它们组成的层次关系。相当于作用于的层次关系，这种层次关系使得公共的动作可以放到基类的控制器里面。

因为它们的责任几乎重叠，AngularJS里的控制器和ASP.NET的WebForms实现方式很相似。

```HTML
<!doctype html>
<html>
  <head>
  </head>
  <body ng-controller="MainCtrl">
    <div ng-controller="ChildCtrl">
      <span>{{user.name}}</span>
      <button ng-click="click()">Click</button>
    </div>
  </body>
</html>
```

```JavaScript
function MainCtrl($scope, $location, User) {
  if (!User.isAuthenticated()) {
    $location.path('/unauthenticated');
  }
}

function ChildCtrl($scope, User) {
  $scope.click = function () {
    alert('You clicked me!');
  };
  $scope.user = User.get(0);
}
```

这个例子只是要说明如何通过基控制器来复用逻辑的最基本方式。在生产环境无论如何我都不推荐把权限认证的逻辑放在控制器里。对于不同路由的访问可以在更高层次的抽象里来实现。

`ChildCtrl` 负责处理像点击带有 `"Click"` 标签的按钮并且通过将它绑定到作用域上将模型暴露给视图。

### 其他

#### 模块模式

其实这不是G4里面的一种设计模式，也不是EAA的P中的一种。这是一种Javascript的 传统模式，主要的目的是提供封装和隐藏。

通过使用模块模式你可以基于Javascript的函数级作用域获得隐藏性。每个模块可以有一个或者多个隐藏在函数作用域里面的成员属性。这个方法返回一个暴露指定模块公共API的对象：

```javascript
var Page = (function () {

  var title;

  function setTitle(t) {
    document.title = t;
    title = t;
  }

  function getTitle() {
    return title;
  }

  return {
    setTitle: setTitle,
    getTitle: getTitle
  };
}());
```
上面的例子我们创造了一个IIFE（立即执行函数表达式），在被调用后返回一个有两个方法 `setTitle` 和 `getTitle` 的对象。返回的对象被赋值给 `Page` 变量。

在这个例子里， `Page` 对象的使用者没有直接访问 `title` 变量，该变量被定义在立即执行函数产生的作用域里。

模块模式在定义AngularJS里的服务时非常有用。使用这种模式我们可以模拟（实际获得了）私有性：

```javascript
app.factory('foo', function () {

  function privateMember() {
    //body...
  }

  function publicMember() {
    //body...
    privateMember();
    //body
  }

  return {
    publicMember: publicMember
  };
});
```

一旦我们想要在其他的组件里注入 `foo` ，我们就只能用共有方法，而无法使用私有方法。这种解决方案在构建一个可以复用库的时候非常好用。

### 数据映射器

>数据映射器是在数据持久层（通常是关系型数据库）和内存中的数据表征（领域模型层）直接进行双向数据转换的数据访问层。这种模式的目的是保持内层内数据表征和数据持久层以及数据映射器彼此独立。

![数据映射器](https://rawgit.com/mgechev/angularjs-in-patterns/master/images/data-mapper.svg "Fig. 10")

像上面描述的一样，数据映射器用来在数据持久层和内存内数据表征之间做数据转换。通常我们的AngularJS应用和服务器端的API服务器通信，服务器接口都是用服务器端语言实现的（Ruby, PHP, Java, Javascript等等）。

通常如果我们有Restful接口 `$resource` 会帮我们用活动对象跟服务器通信。尽管在有些应用中服务器反悔的实体不是那么符合我们前端需要的格式。

举个例子，让我们假定应用中每个用户都有：

- 名字
- 地址
- 朋友列表

我们的API有这些方法：

- `GET /user/:id` - 返回指定用户的名字和地址
- `GET /friends/:id` - 返回指定用户的朋友列表

一种解决方案是我们使用不同的服务，其中一个为第一个方法使用，另外一个为第二个使用。可能更加有用的方法会是如果我们有个单独的叫 `User` 的服务在我们请求用户信息的时候把用户朋友加载进来：

```javascript
app.factory('User', function ($q) {

  function User(name, address, friends) {
    this.name = name;
    this.address = address;
    this.friends = friends;
  }

  User.get = function (params) {
    var user = $http.get('/user/' + params.id),
        friends = $http.get('/friends/' + params.id);
    $q.all([user, friends])
    .then(function (user, friends) {
      return new User(user.name, user.address, friends);
    });
  };
  return User;
});
```

通过这种方式我们创建了一个伪数据映射器，根据我们单页应用的需要适配我们的API。

我们可以这样使用 `User` 服务：

```javascript
function MainCtrl($scope, User) {
  User.get({ id: 1 })
  .then(function (data) {
    $scope.user = data;
  });
}
```

下面的页面模板：

```html
<div>
  <div>
    Name: {{user.name}}
  </div>
  <div>
    Address: {{user.address}}
  </div>
  <div>
    Friends with ids:
    <ul>
      <li ng-repeat="friend in user.friends">{{friend}}</li>
    </ul>
  </div>
</div>
```

## 参考

1. [百科](https://en.wikipedia.org/wiki). The source of all brief descriptions of the design patterns is wikipedia.
2. [AngularJS官方文档](https://docs.angularjs.org)
3. [AngularJS github仓库](https://github.com/angular/angular.js)
4. [页面控制器](http://msdn.microsoft.com/en-us/library/ff649595.aspx)
5. [企业应用架构模式(P of EAA)](http://martinfowler.com/books/eaa.html)
6. [使用依赖注入来减少单例](http://googletesting.blogspot.com/2008/05/tott-using-dependancy-injection-to.html)
7. [为什么要用在JS或者jQuery观察/订阅模式?](https://stackoverflow.com/questions/13512949/why-would-one-use-the-publish-subscribe-pattern-in-js-jquery)