# CORS

跨域资源访问是构建在 HTTP 协议上的一种标准, 该标准规定如何访问跨域资源.

总的来说 CORS 在跨域获取资源的时候, 添加了一些头部, 包括请求头与响应头部. 通过头部的协商来决定是否读取资源.

CORS 是一个协商的流程, 但是具体的协商手段有两种, 取决于请求的类型.

- 简单请求类型
- 非简单请求类型

非简单请求类型在发起资源请求前需要通过 options 发起预检请求, 而简单请求不需要.

# 简单请求

如何判断一个请求是否为简单请求? 在下面的三类条件每一类必须满足一个条件:

请求方法是下列类型:

- GET
- HEAD
- POST

不能人为的设置下列以外的请求头 [规范地址](https://fetch.spec.whatwg.org/#cors-safelisted-request-header):

- Accept
- Accept-Language
- Content-Language
- Content-Type
- DPR
- Downlink
- Save-Data
- Viewport-Width
- Width

Content-Type 值为下面三者之一:

- text/plain
- multipart/form-data
- application/x-www-form-urlencoded

还有以下的两个额外条件:

- 请求中的任意[`XMLHttpRequestUpload`](https://developer.mozilla.org/zh-CN/docs/Web/API/XMLHttpRequestUpload) 对象均没有注册任何事件监听器；[`XMLHttpRequestUpload`](https://developer.mozilla.org/zh-CN/docs/Web/API/XMLHttpRequestUpload) 对象可以使用 [`XMLHttpRequest.upload`](https://developer.mozilla.org/zh-CN/docs/Web/API/XMLHttpRequest/upload) 属性访问。

- 请求中没有使用 [`ReadableStream`](https://developer.mozilla.org/zh-CN/docs/Web/API/ReadableStream) 对象。

可以通过观察简单请求报文的方式快速的了解, 简单请求是如何工作的:

```
GET /resources/public-data/ HTTP/1.1
Host: bar.other
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-us,en;q=0.5
Accept-Encoding: gzip,deflate
Accept-Charset: ISO-8859-1,utf-8;q=0.7,*;q=0.7
Connection: keep-alive
Origin: http://foo.example
```

实际上只有以下的内容是关键信息:

```
GET /resources/public-data/ HTTP/1.1
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-us,en;q=0.5
```

另外其他的请求头实际上是由浏览器添加的, 不要忘记了 CORS 规范所提供的限制实际上是由浏览器来控制的, 但是 HTTP 协议并没有对请求进行限制.

另外浏览器还会附加一些其他有用的字段, 这些字段用户不可以设置但浏览器会自动添加:

```
Origin: http://foo.example
```

告诉了当前页面所在的源.

对于简单请求而浏览器关键的响应头只有一个:

```
Access-Control-Allow-Origin: *
```

表明，该资源可以被**任意**外域访问。如果服务端仅允许来自 http://foo.example 的访问，该首部字段的内容如下：

```
Access-Control-Allow-Origin: http://foo.example
```

# 预检请求

简单来讲, 浏览器是这样处理非简单请求的:

1. 通过 OPTIONS 请求去询问浏览器, 这个环节被称为预检请求
2. 询问是否接收非简单请求规则外的内容
3. 服务器同意响应 200 且返回服务器支持的规则

符合以下规则的请求, 需要发送预检请求:

使用了下面任一 HTTP 方法：  

- [`PUT`](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Methods/PUT)
- [`DELETE`](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Methods/DELETE)
- [`CONNECT`](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Methods/CONNECT)
- [`OPTIONS`](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Methods/OPTIONS)
- [`TRACE`](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Methods/TRACE)
- [`PATCH`](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Methods/PATCH)

人为设置了[对 CORS 安全的首部字段集合](https://fetch.spec.whatwg.org/#cors-safelisted-request-header)之外的其他首部字段。该集合为：  

- [`Accept`](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/Accept)
- [`Accept-Language`](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/Accept-Language)
- [`Content-Language`](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/Content-Language)
- [`Content-Type`](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/Content-Type) (需要注意额外的限制)
- `DPR`
- `Downlink`
- `Save-Data`
- `Viewport-Width`
- `Width`

[`Content-Type`](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/Content-Type) 的值不属于下列之一:  

- `application/x-www-form-urlencoded`
- `multipart/form-data`
- `text/plain`

以及:

- 请求中的[`XMLHttpRequestUpload`](https://developer.mozilla.org/zh-CN/docs/Web/API/XMLHttpRequestUpload) 对象注册了任意多个事件监听器。

- 请求中使用了[`ReadableStream`](https://developer.mozilla.org/zh-CN/docs/Web/API/ReadableStream)对象。

再次我们尝试使用报文去解释预检请求的完整流程, 首先发送预检请求:

```
OPTIONS /resources/access-control-with-post-preflight/ HTTP/1.1
Origin: http://arunranga.com
Access-Control-Request-Method: POST
Access-Control-Request-Headers: X-PINGARUNER
```

这里删除了不太重要的请求头, 保留了我们需要知道的几个:

- `Origin: http://arunranga.com` 告诉服务器我们的源
- `Access-Control-Request-Method: POST` 告诉服务器我们会进行 POST 请求
- `Access-Control-Request-Headers: X-PINGARUNER` 告诉服务器我们将使用一个特殊的请求头 `X-PINGARUNER` 它不在简单请求约束的范围内.

这里显示的是支持这次预检请求的服务器的响应头的简化版本:

```
HTTP/1.1 200 OK
Access-Control-Allow-Origin: http://arunranga.com
Access-Control-Allow-Methods: POST, GET, OPTIONS
Access-Control-Allow-Headers: X-PINGARUNER
Access-Control-Max-Age: 1728000
```

- `Access-Control-Allow-Origin: http://arunranga.com` 告诉浏览器服务器支持的源
- `Access-Control-Allow-Methods: POST, GET, OPTIONS` 告诉浏览器服务器支持的请求方法
- `Access-Control-Allow-Headers: X-PINGARUNER` 告诉浏览器服务器支持的特殊请求头
- `Access-Control-Max-Age: 1728000` 告诉浏览器这次预检请求的有效时间, 从浏览器预检请求响应算起直到经过 1728000 秒后浏览器可以再次发起预检请求

# 附带凭证

对于嵌入的跨域内容是否附带 cookie, 这取决于 CSP 和 cookie 写入时候的设定.

对于 XHR 和 fetch 请求来说默认是不可以携带凭证的, 也就是说你不能通过 XHR 和 fetch 来完成 HTTP 协议的:

- cookie
- Authorization

操作.

但是 CORS 协议提供了附带凭证的手段即在请求的时候提供一个参数选项, 即将 `withCredentials` 设置为 true, 在 XHR 和 fetch 中都可以开启:

```javascript
var invocation = new XMLHttpRequest();
var url = 'http://bar.other/resources/credentialed-content/';
    
function callOtherDomain(){
  if(invocation) {
    invocation.open('GET', url, true);
    invocation.withCredentials = true;
    invocation.onreadystatechange = handler;
    invocation.send(); 
  }
}
```

在本例中的 cookie 便可以发送的服务端, 但是浏览器是否把服务器响应的数据交由脚本, 着取决于服务器端是否需要这个凭证, 我们来看一下响应报文:

```
HTTP/1.1 200 OK
Access-Control-Allow-Origin: http://foo.example
Access-Control-Allow-Credentials: true
```

- `Access-Control-Allow-Credentials: true` 告诉浏览器服务器接收凭证认证

浏览器知道了这一点才会将数据交由脚本.

另外一点值得注意的是, 对于附带身份凭证的请求，服务器不得设置 `Access-Control-Allow-Origin` 的值为 "*", 必须明确指定接收的源.

以及在预检请求中服务器同样可以响应 `Access-Control-Allow-Credentials: true` 也就是提前告诉了浏览器服务器是否接收凭证.