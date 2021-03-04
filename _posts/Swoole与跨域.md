---
title: Swoole与跨域
date: 2018-10-07 19:29:55
tags: 
- PHP
categories:
- PHP
---

## Swoole

[Swoole](https://wiki.swoole.com/)是我经常在使用的PHP的协程高性能网络通信引擎，非常好用，为PHP又提供了许多出乎意料的使用拓展，如携程异步等。并且是由`C/C++`语言编写，作为一个扩展安装使用非常的简单。

我在工作之余尝试将`Swoole`和公司框架`Gini`进行共同使用，效果不错，可见两者都符合低耦合的设计理念。简单的增加了一个`index.php`来作为程序的主入口，就非常简单的使用起了大部分功能。

例如在接受`http`请求方面:

```php
public function request($req, $res) {
    $header = $req->header;
    $_SERVER = array_merge($_SERVER, $req->server);

    // TODO: 传入res对象使后续可以对res对象进行操作
    $uri = trim($_SERVER['request_uri'], '/');
    $result = \Gini\CGI::request($uri, [
        'header' => $header,
        'get' => $req->get,
        'post' => $req->post,
        'files' => [], // 暂且先不考虑file
        'route' => $uri,
        'method' => $_SERVER['request_method'],
        'swoole' => $this->server, // swoole_server 对象
        'raw' => $req->rawContent(),
    ])->execute();

    // 防止php://output过多挤压supervisor.log
    ob_start();
    $result->output();
    ob_end_clean();

    $res->status(http_response_code());
    $res->end(J($result->content()));
}
```

就非常简单的可以将`Swoole`传递过来的请求，分发到`Gini`框架的路由当中去，再做各自的逻辑处理。

但是当我部署到生产环境的时候，就出现了一件比较尴尬的事情，跨域警告了。

## 跨域

什么是跨域呢，前端可能接触的比较多，这并不是服务器做出的限制，而是浏览器的安全策略。

浏览器为了防止不同域之间发生不安全的数据交互或者攻击，会默认阻止这种行为。但是有些时候我们的应用的确是部署在不同的域下，这时候我们就要了解跨域的原理了。

当我们打开调试工具的时候，会发现在跨域请求之前，我们会看到一个类似这样的请求

``` 
Request URL: http://path.to/api
Request Method: OPTIONS
```

但是没有任何的消息体进行返回，然后就报错了，不能跨域。

实际上，浏览器在阻止跨域请求之前，会向目标服务器发送一个这样的请求，来询问服务器：什么样子的请求可以跨域啊？

然后服务器就会将自己的配置告诉浏览器，来方便做后续的判断。这也就是网上大部分的解决方案有效的原因了，比如`nginx`:

``` nginx
add_header Access-Control-Allow-Origin http://path.to;
add_header Access-Control-Allow-Credentials true;
add_header Access-Control-Allow-Headers X-Requested-With;
add_header Access-Control-Allow-Methods GET,POST,OPTIONS;
```

实际上就是告诉了浏览器，我允许跨域的`Origin`, `Headers`, `Methods`等，如果请求符合，则就可以进行正常的跨域请求。

## 解决

这个在传统的`PHP`开发中很好解决，我们一般会在Web服务器如`nginx`或`apache`中进行修改解决，但是`Swoole`实际上自己充当了一部分`Http`服务器的角色，所以我们需要稍微做一下改动。

上面说到了，具体能否跨域，主要取决于`OPTIONS`请求所返回的信息，所以我们可以在这里加以拦截:

``` php
public function request($req, $res) {
    if ($req->server['request_method'] == 'OPTIONS') {
        $res->status(http_response_code());
        $res->end();
        return;
    }
    ...
}
```

这样就可以终止后面的路由解析行为，防止后面的逻辑再做其他的处理，然后像nginx配置的那样，我们需要加上我们希望返回的头内容：

``` php
public function request($req, $res) {
    // 跨域OPTIONS返回
    $res->header('Access-Control-Allow-Origin', '*');
    $res->header('Access-Control-Allow-Methods', 'GET, POST, DELETE, PUT, PATCH, OPTIONS');
    $res->header('Access-Control-Allow-Headers', 'Authorization, User-Agent, Keep-Alive, Content-Type, X-Requested-With');
    if ($req->server['request_method'] == 'OPTIONS') {
        $res->status(http_response_code());
        $res->end();
        return;
    }
```

这里我简单的加入了几个常用的`Methods`和`Headers`，这样实际上我们`Swoole`在接收到浏览器的跨域侦测时，就能返回正常的信息并继续跨域请求了。