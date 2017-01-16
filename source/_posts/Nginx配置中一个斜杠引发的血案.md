Nginx经常被用来做前端的代理服务器，代理相关配置好之后，需要配置proxy_pass，最近因为一个"/"引发了一系列的问题，记录一下．

proxy_pass用来配置代理后端的服务器地址，可以带上端口号，有两种写法
- **不带 `/` :** 根路径`/`会被保留，也就是 http://localhost/test/page, 后端获取到的是完整的路径`/test/page`

``` sh
location /test {
    proxy_pass http://127.0.0.1:81
}
```
- **带 `/` :** 根路径`/`会被吃掉，也就是 http://localhost/test/page, 后端获取到的是完整的路径`/page`

``` sh
location /test {
    proxy_pass http://127.0.0.1:81/
}
```

大部分情况代理或者仅仅是做子站隔离的时候都是需要保留根路径的（特别是子站隔离很多时候就是用根路径的不同来区分的），所以第一种情况用的会更多一点，配置proxy_pass后根路径会被保留．
# 顺便一记

发现该问题是在使用Yii2.0项目开发出现的．一般我们都会配置urlManager, 正常配置一些rules，如下所示

``` php
'components' => [
       ...
        'urlManager' => [
            'enablePrettyUrl' => true,
            'enableStrictParsing' => true, //注意这里
            'showScriptName' => false,
            'rules' => [
                'webapp/<module:\w+>/<controller:[\w-]+>/<action:[\w-]+>/<id:[\w\d,]{24}(,[\w\d]{24})*>'       => '<module>/<controller>/<action>',
                'webapp/<module:\w+>/<submodule:\w+>/<controller:[\w-]+>/<action:[\w-]+>/<id:[\w\d,]{24}>'     => '<module>/<submodule>/<controller>/<action>',
                'webapp/<module:\w+>/<controller:[\w-]+>/<action:[\w-]+>'                                      => '<module>/<controller>/<action>',
                'webapp/<module:\w+>/<submodule:\w+>/<controller:[\w-]+>/<action:[\w-]+>'                      => '<module>/<submodule>/<controller>/<action>',
            ],
        ],
       ...
```

这里添加了`enableStrictParsing`的配置，如果设置为true则表示所有的路由匹配只能在rules的列表里面，这要不是列表中的配置项就返回404（其实也未可厚非），但关键是要能够匹配到啊！

因为之前配置的问题，使用了上面提到的第二种配置方式，根路径被吃掉了．一个请求过来会先做路由解析，到`/yii2/web/Request.php`里面的`resolve`方法

``` php
public function resolve()
{
    $result = Yii::$app->getUrlManager()->parseRequest($this);
    if ($result !== false) {
        list ($route, $params) = $result;
        $_GET = array_merge($_GET, $params);

        return [$route, $_GET];
    } else {
        throw new NotFoundHttpException(Yii::t('yii', 'Page not found.'));
    }
}
```

在解析路由中会使用我们之前配置的`urlManager`实例上的`parseRequest`方法，在`/yii2/web/UrlManager.php`

``` php
public function parseRequest($request)
{
    if ($this->enablePrettyUrl) {
        $pathInfo = $request->getPathInfo();
        /* @var $rule UrlRule */
       //这里匹配你自己配置的规则
        foreach ($this->rules as $rule) {
            //这里调用了rule对象的parseRequest方法，后面会追到
            if (($result = $rule->parseRequest($this, $request)) !== false) {
                return $result;
            }
        }
　　//这里是强制使用你自己配置生效的地方
        if ($this->enableStrictParsing) {
            return false;
        }

        Yii::trace('No matching URL rules. Using default URL parsing logic.', __METHOD__);
　　//这里是yii自己默认生成的规则
        $suffix = (string) $this->suffix;
        if ($suffix !== '' && $pathInfo !== '') {
            $n = strlen($this->suffix);
            if (substr_compare($pathInfo, $this->suffix, -$n, $n) === 0) {
                $pathInfo = substr($pathInfo, 0, -$n);
                if ($pathInfo === '') {
                    // suffix alone is not allowed
                    return false;
                }
            } else {
                // suffix doesn't match
                return false;
            }
        }

        return [$pathInfo, []];
    } else {
        Yii::trace('Pretty URL not enabled. Using default URL parsing logic.', __METHOD__);
        $route = $request->getQueryParam($this->routeParam, '');
        if (is_array($route)) {
            $route = '';
        }

        return [(string) $route, []];
    }
}
```

调试发现就是我们配置的路由规则没有生效，继续往下找，话说这个rule对象是什么鬼？搜索后发现，rule列表是在init的时候生成的

``` php
public function init()
{
    parent::init();

    if (!$this->enablePrettyUrl || empty($this->rules)) {
        return;
    }
    if (is_string($this->cache)) {
        $this->cache = Yii::$app->get($this->cache, false);
    }
　//根据是否配置了cache在cache里面也放一份
    if ($this->cache instanceof Cache) {
        $cacheKey = __CLASS__;
        $hash = md5(json_encode($this->rules));
        if (($data = $this->cache->get($cacheKey)) !== false && isset($data[1]) && $data[1] === $hash) {
            $this->rules = $data[0];
        } else {
            $this->rules = $this->buildRules($this->rules);
            $this->cache->set($cacheKey, [$this->rules, $hash]);
        }
    } else {
        $this->rules = $this->buildRules($this->rules);
    }
}
```

主要逻辑就是调用`buildRules`生成规则对象，继续看buildRules的实现，发现rule是使用yii的createObject方法，默认用`yii\web\UrlRule`生成规则对象

``` php
protected function buildRules($rules)
{
    $compiledRules = [];
    $verbs = 'GET|HEAD|POST|PUT|PATCH|DELETE|OPTIONS';
    foreach ($rules as $key => $rule) {
        if (is_string($rule)) {
            $rule = ['route' => $rule];
            if (preg_match("/^((?:($verbs),)*($verbs))\\s+(.*)$/", $key, $matches)) {
                $rule['verb'] = explode(',', $matches[1]);
                if (!in_array('GET', $rule['verb'])) {
                    $rule['mode'] = UrlRule::PARSING_ONLY;
                }
                $key = $matches[4];
            }
            $rule['pattern'] = $key;
        }
        if (is_array($rule)) {
　　　// ruleConfig的默认配置如下:
　　　// public $ruleConfig = ['class' => 'yii\web\UrlRule'];
            $rule = Yii::createObject(array_merge($this->ruleConfig, $rule));
        }
        if (!$rule instanceof UrlRuleInterface) {
            throw new InvalidConfigException('URL rule class must implement UrlRuleInterface.');
        }
        $compiledRules[] = $rule;
    }
    return $compiledRules;
}
```

再看`yii\web\UrlRule`的`parseRequest`

``` php
public function parseRequest($manager, $request)
{
    if ($this->mode === self::CREATION_ONLY) {
        return false;
    }

    if (!empty($this->verb) && !in_array($request->getMethod(), $this->verb, true)) {
        return false;
    }

    $pathInfo = $request->getPathInfo();
    $suffix = (string) ($this->suffix === null ? $manager->suffix : $this->suffix);
    if ($suffix !== '' && $pathInfo !== '') {
        $n = strlen($suffix);
        if (substr_compare($pathInfo, $suffix, -$n, $n) === 0) {
            $pathInfo = substr($pathInfo, 0, -$n);
            if ($pathInfo === '') {
                // suffix alone is not allowed
                return false;
            }
        } else {
            return false;
        }
    }

    if ($this->host !== null) {
        $pathInfo = strtolower($request->getHostInfo()) . ($pathInfo === '' ? '' : '/' . $pathInfo);
    }
    //主要看这里，将匹配规则和匹配项目打印出来
    // var_dump([$this->pattern, $pathInfo]); 
    // echo '</br>';
    if (!preg_match($this->pattern, $pathInfo, $matches)) {
        return false;
    }
    foreach ($this->defaults as $name => $value) {
        if (!isset($matches[$name]) || $matches[$name] === '') {
            $matches[$name] = $value;
        }
    }
    $params = $this->defaults;
    $tr = [];
    foreach ($matches as $name => $value) {
        if (isset($this->_routeParams[$name])) {
            $tr[$this->_routeParams[$name]] = $value;
            unset($params[$name]);
        } elseif (isset($this->_paramRules[$name])) {
            $params[$name] = $value;
        }
    }
    if ($this->_routeRule !== null) {
        $route = strtr($this->route, $tr);
    } else {
        $route = $this->route;
    }

    Yii::trace("Request parsed with URL rule: {$this->name}", __METHOD__);

    return [$route, $params];
}
```

将匹配时候的正则表达式和最终匹配结果打印出来

```
array(2) { [0]=> string(103) "#^webapp/(?P\w+)/(?P[\w-]+)/(?P[\w-]+)/(?P[\w\d,]{24}(,[\w\d]{24})*)$#u" [1]=> string(25) "/demo/default/index" } 
array(2) { [0]=> string(108) "#^webapp/(?P\w+)/(?P\w+)/(?P[\w-]+)/(?P[\w-]+)/(?P[\w\d,]{24})$#u" [1]=> string(25) "/demo/default/index" } 
array(2) { [0]=> string(69) "#^webapp/(?P\w+)/(?P[\w-]+)/(?P[\w-]+)$#u" [1]=> string(25) "/demo/default/index" } 
```

可以看到`/webapp`都被吃掉了，这就是之前nginx配置引起的问题了，正则中还有`^webapp`，所以匹配到才怪了呢～
