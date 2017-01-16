在web开发领域，“中间件”代表的是使用应用的一部分来包装原有应用程序，添加额外的功能。这个概念通常未收到充分的赏识，但是我认为还是很棒的。

我认为一个好的中间件有单一的责任，它是可插拔的并且是自包含的。这意味着你可以把它在借口层次嵌入并且马上能用。它不会影响你的编码风格，它也不是一个框架，而仅仅是你请求处理流中的另外一层。没有必要重写你的代码，你决定用中间件，只需要把它加到公式里面，如果你反悔了，就去掉它，就是这么简单。

反观Go语言中，HTTP中间件即使是在标准库里也是很普遍的。尽管开始看起来不是很明显，`net/http`包里面，像[StripPrefix](http://golang.org/pkg/net/http/#StripPrefix)或者[TimeoutHandler](http://golang.org/pkg/net/http/#TimeoutHandler)这些方法，就是我们期待中中间件的样子。它们包装了你的处理器，在处理请求和响应的时候加额外的步骤。

我最近的Go包[nosurf](https://github.com/justinas/nosurf)也是个中间价。我有意从最开始设计这个包。大部分情况，你不需要在应用程序层面注意到CSRF的检查：nosurf像其他合格的中间件一样，它是独立于系统的并且能够跟其他使用标准`net/http`接口的工具很好的协作。

你也可以用中间件来：

* 通过长度隐藏来减轻[BREACH 攻击](http://netsecurity.51cto.com/art/201308/406091.htm)
* 请求速率控制
* 抵挡恶意木马
* 提供调试信息
* 添加[HSTS](https://en.wikipedia.org/wiki/HTTP_Strict_Transport_Security), [X-Frame-Options](https://developer.mozilla.org/en-US/docs/Web/HTTP/X-Frame-Options) 报头
* 优雅地从panic中恢复
* ...还有其他可能的功能

# 写一个简单的中间件
第一个例子我们来写个只允许用户在单一域名（在HTTP协议的Host报头中指定）下访问我们的网站的中间件。这样的中间件可以保护web应用免受[host欺骗](http://www.skeletonscribe.net/2013/05/practical-http-host-header-attacks.html)的攻击。

## 构造类型

首先让我们来位这个中间件来定义一种类型，我们姑且叫它 `SingleHost`。

```go
type SingleHost struct {
    handler     http.Handler
    allowedHost string
}
```

它只包含了两个字段：

* 被包装的处理器，我们会让带有合法Host的请求调用它
* 被允许访问的host的值

因为我们使用小写来定义字段名，让这些字段对于在我们的包里面是私有的，我们还需要给我们的类型定义一个构造器。

```go
func NewSingleHost(handler http.Handler, allowedHost string) *SingleHost {
    return &SingleHost{handler: handler, allowedHost: allowedHost}
}
```

## 处理请求

现在是实际的逻辑代码。为了实现 `http.Handler` 接口，我们的类型只需要有一个方法：

```go
type Handler interface {
        ServeHTTP(ResponseWriter, *Request)
}
```

实现如下：

```go
func (s *SingleHost) ServeHTTP(w http.ResponseWriter, r *http.Request) {
    host := r.Host
    if host == s.allowedHost {
        s.handler.ServeHTTP(w, r)
    } else {
        w.WriteHeader(403)
    }
}
```

`ServeHTTP` 方法仅仅检查请求中的Host报头：

* 如果请求匹配构造函数中设置的`allowedHost`值，它就会调用被包装处理器上的`ServeHTTP`方法，这样处理请求的责任就传递下去了。
* 如果不匹配，请求返回403（未授权）状态码，请求处理到此为止。

在后面一种情况，原来处理器中的`ServeHTTP`方法永远不会被调用到，处理器甚至不知道有这么一个请求发出过。

现在我们的中间件已经写完了。我们只需要把它嵌入应用。我们在中间件里包裹处理器，而不是直接把我们原来的处理器传递给`net/http`服务器。

```go
singleHosted = NewSingleHost(myHandler, "example.com")
http.ListenAndServe(":8080", singleHosted)
```

## 另外一种方式

我们上面实现的中间件真的很简单：它只有15行代码。写这种中间件，有一个用更少代码量模板来实现的方法。由于Go语言对于函数一等公民和闭包的支持，以及有简洁的`http.HandlerFunc`包装器，我们可以把这些逻辑作为一个简单的方法来实现而不是一个独立的结构体类型。下面是我们中间价的函数式版本完整实现。

```go
func SingleHost(handler http.Handler, allowedHost string) http.Handler {
    ourFunc := func(w http.ResponseWriter, r *http.Request) {
        host := r.Host
        if host == allowedHost {
            handler.ServeHTTP(w, r)
        } else {
            w.WriteHeader(403)
        }
    }
    return http.HandlerFunc(ourFunc)
}
```

这里我们定义了一个简单的函数叫做`SingleHost`，这个函数可以接受一个需要被包装的处理器和被允许的Host名称。在方法里我们构造了一个和之前版本中`ServeHTTP`相似的方法。我们内部的方法实际是一个闭包，所以它能够访问外部的变量。最终，`HandlerFunc`让我们能够把这个方法作为一个`http.Handler`来使用。

是使用`HandlerFunc`还是使用你自己的`http.Handler`类型最终是由你来决定的。对于基本的情况，一个方法就够了，如果你发现你的中间件越来越臃肿，你就会想要定义自己的结构体类型并且分离逻辑到多个方法中去。

同时，标准库实际会用到上面两种方法来创造中间件。`StripPrefix`是一个返回`HandlerFunc`的方法，而`TimeoutHandler`虽然是个方法，但是返回的是个处理请求的自定义结构体类型。

# 一个更复杂的情况

我们的`SingleHost`中间件没啥实用价值：我们只是检查了请求中的某个属性然后把请求传递给原始的处理器，既没有再关注它也没有返回一个我们自己的响应，并且让原始的处理器获取到它。更不用说，有些情况是不光要基于请求来做出动作，我们的中间件还可能需要在原来的处理器写入响应之后再进一步通过某种方式处理返回给客户端的响应。

## 添加数据很简单

如果你想要在被包装的处理器已经写入响应之后再添加数据，你只需要在完成的时候调用`Write()`方法：

```go
type AppendMiddleware struct {
    handler http.Handler
}
```

```
func (a *AppendMiddleware) ServeHTTP(w http.ResponseWriter, r *http.Request) {
    a.handler.ServeHTTP(w, r)
    w.Write([]byte("Middleware says hello."))
}
```

现在响应会在原有处理器输出的任何内容之后添加中间件输出的hello。

## 问题

处理其他类型的响应就有点麻烦了。假如我们想要在数据添加在放回响应的头里面。如果我们在原始的处理器前调用`Write()`方法就会对状态码和报头失去控制，因为第一次`Write()`调用立即将它们写入。

通过其他任何方式（比如，替换字符串），改变某个响应的报头或者设置一个不同的状态码不会起作用，这是因为一个相似的原因：当被包装的处理器返回之后，上面这些内容已经被返回给客户端了。

为了应对这种情况，我们需要一种特殊的`ResponseWriter`能够像缓冲区一样工作，手机响应并且为了之后使用保存数据（以及修改）。我们之后会把`ResponseWriter`缓存传递给原始的处理器而不是给它真实的RW，这样可以防止它直接讲响应返回。

幸运的是，在Go的标准库中有这样的工具。`net/http/httptest`包中的`ResponseRecorder`做了所有我们需要的：它保存响应状态码，报头的映射表并且把报文积攒到缓冲流中。尽管它本来是用于测试的（就像包名表达的一样），但是正好也满足了我们的情况。

为了内容的完整性，让我们来看个使用`ResponseRecorder`修改响应中东西的例子。

```go
type ModifierMiddleware struct {
    handler http.Handler
}

func (m *ModifierMiddleware) ServeHTTP(w http.ResponseWriter, r *http.Request) {
    rec := httptest.NewRecorder()
    // passing a ResponseRecorder instead of the original RW
    m.handler.ServeHTTP(rec, r)
    // after this finishes, we have the response recorded
    // and can modify it before copying it to the original RW

    // we copy the original headers first
    for k, v := range rec.Header() {
        w.Header()[k] = v
    }
    // and set an additional one
    w.Header().Set("X-We-Modified-This", "Yup")
    // only then the status code, as this call writes out the headers 
    w.WriteHeader(418)

    // The body hasn't been written (to the real RW) yet,
    // so we can prepend some data.
    data := []byte("Middleware says hello again. ")

    // But the Content-Length might have been set already,
    // we should modify it by adding the length
    // of our own data.
    // Ignoring the error is fine here:
    // if Content-Length is empty or otherwise invalid,
    // Atoi() will return zero,
    // which is just what we'd want in that case.
    clen, _ := strconv.Atoi(r.Header.Get("Content-Length"))
    clen += len(data)
    r.Header.Set("Content-Length", strconv.Itoa(clen))

    // finally, write out our data
    w.Write(data)
    // then write out the original body
    w.Write(rec.Body.Bytes())
}
```

下面是我们通过中间件返回"Success!"包装过的处理器响应。

```sh
HTTP/1.1 418 I'm a teapot
X-We-Modified-This: Yup
Content-Type: text/plain; charset=utf-8
Content-Length: 37
Date: Tue, 03 Sep 2013 18:41:39 GMT

Middleware says hello again. Success!
```

这为我们开启了新的可能性。被包装的处理器完全掌握在我们手里：即使是在请求被处理完之后，我们也能以任何我们想要的方式操纵响应。

## 跟其他处理器共享数据

在很多情况下，你的中间件需要暴露某些信息给其他的中间件或者你自己的应用。举个例子，nosurf需要给用户提供访问CSRF token的方法和失败的理由（如果有的话）。

一种优雅的方式是使用映射表，通常是存在一个隐藏在包内的全局变量里，用来映射`http.Request`指针和需要的数据片段，同时暴露包级别（或者处理器级别）的方法来访问数据。

我在nosurf中也使用了这种模式。这里，我创建了一个全局的映射表。注意到这里需要一个互斥锁，因为Go语言的映射表默认不是线程安全的，参考[这里](http://blog.golang.org/go-maps-in-action#TOC_6.)。

```go
type csrfContext struct {
    token string
    reason error
}

var (
    contextMap = make(map[*http.Request]*csrfContext)
    cmMutex    = new(sync.RWMutex)
)
```

数据在处理器中被设置进去，通过包导出的`Token()`方法把数据暴露出来。

```go
func Token(req *http.Request) string {
    cmMutex.RLock()
    defer cmMutex.RUnlock()

    ctx, ok := contextMap[req]
    if !ok {
            return ""
    }

    return ctx.token
}
```

你可以在nosurf库的[context.go](https://github.com/justinas/nosurf/blob/master/context.go)中找到完成的实现代码。

相比于我选择自己位nosurf自己造轮子，有个[gorilla/context](http://www.gorillatoolkit.org/pkg/context)的包实现了一个通用的映射表来保存请求的信息。在大部分情况下，它能够满足你的需要，让你不需要自己实现一个共享的存储结构以及踩人家踩过的坑。它甚至提供在处理完之后清除请求数据的[中间件](http://www.gorillatoolkit.org/pkg/context#ClearHandler)。

# 总结

这篇文章的目的是是给Go语言开发者描述中间件的概念并且展示一些基本的Go语言里面实现中间件的方式。作为一个相对年轻的语言，Go语言提供了非常好的标准HTTP接口。这也是让编写Go的中间件更加方便甚至有趣的因素。

然而，仍然缺乏高质量的Go语言的HTTP工具。大部分但不是所有上面我提到关于Go的中间件的想法还没有实现。现在你知道怎么在Go语言中实现中间件了，为什么不自己整个呢？:-)

P.S. 你可以在github的[gist](https://gist.github.com/justinas/7059324)找到文中相关的实现代码。

# 后记

对于中间件的实现最常见的思路是使用`责任链`的方式，很多框架实现中都是以filter的概念提供，一级的事情做完传给下一级。最原始Go语言中使用`装饰器`来实现同样的效果，通过预先定义包装顺序来控制处理流程的顺序。通过在内存中提供公共内存区域来在不同的中间件之间共享数据。其实这种方法中间件逐渐增多之后会难以维护，你会看到这样的code：

```go
Middleware1(Middleware2(Middleware3(App)))
```

本文的作者实现了一个简单的责任链式的[库](https://github.com/justinas/alice)，也写[文章](https://justinas.org/alice-painless-middleware-chaining-for-go/)讲解了下用例，将上面的代码转化为

```go
alice.New(Middleware1, Middleware2, Middleware3).Then(App).
```

本文只能说是简单介绍了Go语言中中间件实现的思路，未必是最好的实现，这个见仁见智把，最后，本文的英文原文看这里：[传送门](https://justinas.org/writing-http-middleware-in-go/)
