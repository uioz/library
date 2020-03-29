# Cookie 与同源策略

当一个链接指向到一个网站的时候, 用户点击链接, 服务器可以使用 `Set-Cookie` 响应头来写入 cookie.

在随后的请求过程中, 服务器写入的 cookie 会被浏览器通过 `cookie` 字段以及对应的值写入到请求头中.

一个 HTML 页面可以引入多种资源. 根据同源策略(Same-Origin Policy), 对资源的引用分为了三种:

- 跨域写操作, 例如 form 表单中的 action 属性可以是跨域地址.
- 跨域资源嵌入, `<link>` `<script>` `<audio>` `<video>` `<img>` `<iframe>` `@font-face` ....
- 跨域读操作, XHRHttpRequest fetch 等在脚本内部发起的请求

实际上跨域资源嵌入和跨域读操作都是对资源的读取, 如何区分这两点呢, 我个人认为 "跨域资源嵌入" 都存在 HTML 中对应的标签, 而 "跨域读操作" 是不需要的.

上述分类描述了页面中所有的请求, 而在一个页面中的所有的资源读取, 基于 HTTP 协议的情况下, 服务端都可以通过 `Set-Cookie` 写入 cookie.

这里之所以指出这一点就是请记住, 页面可以引用跨域的资源, 这些资源对应的服务器是可以写入 cookie 的. cookie 并没有和当前的页面进行绑定, cookie 存在一块单独的存储区域中它们是集中存储的.

举一个例子, 存在三个网站(忽略细节):

- A.com
- B.com
- C.org

A 和 B 以及 C 被引用的静态资源都会在访问的时候写入 cookie, A 和 B 都引用了 C 中的静态资源, 进入 A 网站:

- 写入 A 网站的 cookie
- 写入 C 网站的静态资源的 cookie

关闭 A 网站, 进入 B 网站:

- 写入 B 网站的 cookie
- **更新** C 网站的静态资源的 cookie (之所以说是更新是因为 B 页面在未请求 C 静态资源之前, 就已经可以访问它之前写入的 cookie 的, 而重复写入相当于更新)

# cookie 的作用域

显然你需要控制 cookie 的作用范围, 而 cookie 是随着请求一并发送的, 我们的作用范围实际是用于 "控制 cookie 能否随着请求而发送". HTTP 提供了两方面来控制 cookie 的作用范围:

- cookie 必须在给定的域名下才可以被发送
  - 对应的 `Set-Cookie` 选项是 Domain
- cookie 必须在给定的路径下才可以被发送
  - 对应的 `Set-Cookie` 选项是 Path

Domain 可以省略不设置, 在这种情况下, Domain 被客户端自动设置为当前的源, 也就是页面对应的地址, 例如当前的地址为 `dev.xxx.com` 自动设置的是 `xxx.com` 默认的设置是不包含子域名的.

Path 则非常简单, `/` 匹配所有路径, `/xxx` 匹配 `/xxx` 以及 `/xxx/abc` 等以 `/xxx` 开头的其他路径.

# cookie 生命周期

从是否可以被指定生命周期的角度来将可以分为两种:

- 指定时间后使 cookie 无效
  - 会称为 "持久性 cookie"
- 会话结束后使 cookie 无效
  - 被称为 "会话期 cookie"

会话期 cookie 不用做任何设置也是默认行为, 当 cookie 被写入后, 一旦浏览器关闭此类 cookie 会被自动销毁.

持久性 cookie 通过 `Set-Cookie` 的两个选项来控制:

- Expires 对应的是一个具体时间, 浏览器会检查 cookie 是否超过了给定的时间而取消 cookie 发送.
- Max-Age 对应的是一个最大过期时间以秒计算, 浏览器会检查 cookie 根据 cookie 接收时间和发送时间的差是否大于最大过期时间来判断.

**PS**: 这两个属性和 HTTP 响应头中的规则一致, 可填写的参数也一致.

# 安全控制

cookie 本身很不安全, 本身非常容易泄露以及窃取, 在常见的 web 安全问题中很多都和 cookie 相关:

- XSS (跨站脚本攻击)
- CSRF (跨站请求伪造)

而 cookie 也提供了简单的安全控制, 这些机制作为 `Set-Cookie` 属性出现.

Secure 属性要求只有 HTTPS 地址才可以发送这个 cookie, 功能非常简单, 但是需要注意的是, 你不应该将敏感信息存放到 cookie 中, 即使你开启了这个选项, 黑客依然可以通过其他手段来获取到这个 cookie. Secure 的功能仅仅是让 cookie 享受 HTTPS 的技术红利而已.

HttpOnly 属性控制 JavaScript 通过 `document.cookie` 来读取 cookie, 也就是说 `Set-Cookie` 如果指定这个 cookie 为 HttpOnly 那么 JavaScript 就无法获取到它. 使用它可以避免 XSS 攻击的威胁因为 cookie 无法从脚本中被泄露出去, 一般用于 session 的处理.

SameSite 是比较新的属性, Chrome 52 和 Firefox 52 后才正式支持, 现在以及可以安全使用. 它关注点在是否允许 "跨站请求" 发送 cookie, 该选项可以减弱 CSRF 的威胁.

所谓的 "跨站请求" 是指, 在当前域下访问其他域的资源, 是否发送对应域的 cookie.

这个属性提供了三种值:

- None 相当于不指定, 允许跨站携带 cookie

- Strict 不允许非当前域下的资源在请求的时候携带 cookie

- Lax 允许导航到目标网址的 GET 请求携带 cookie (现在是浏览器的默认策略)

  - | 请求类型  |                 示例                 |        None | Lax         |
    | --------- | :----------------------------------: | ----------: | ----------- |
    | 链接      |         `<a href="..."></a>`         | 发送 Cookie | 发送 Cookie |
    | 预加载    | `<link rel="prerender" href="..."/>` | 发送 Cookie | 发送 Cookie |
    | GET 表单  |  `<form method="GET" action="...">`  | 发送 Cookie | 发送 Cookie |
    | POST 表单 | `<form method="POST" action="...">`  | 发送 Cookie | 不发送      |
    | iframe    |    `<iframe src="..."></iframe>`     | 发送 Cookie | 不发送      |
    | AJAX      |            `$.get("...")`            | 发送 Cookie | 不发送      |
    | Image     |          `<img src="...">`           | 发送 Cookie | 不发送      |



# 参考

> https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/Set-Cookie
>
> https://zhuanlan.zhihu.com/p/110220391
>
> http://www.ruanyifeng.com/blog/2019/09/cookie-samesite.html
>
> https://developer.mozilla.org/zh-CN/docs/Web/Security/Same-origin_policy