# onload 和 DOMContentLoad 事件之间的区别

通常大家说 DOMContentLoad (DCL) 时文档解析完成即 DOM 解析完成后便触发.

但是实际更复杂一些, 或者说上面丢失了一些细节.

通过 chrome 的开发者工具中的性能面板我们来观察一个"地址", 这个地址返回的 HTML 文件中包含了以下的主要静态内容:

```html
<link href="/static/build/styles/react-header.b8f80567be6b.css" rel="stylesheet" type="text/css" />
<link href="/static/build/styles/auth-modal.af9dfc7ec717.css" rel="stylesheet" type="text/css" />
<link href="/static/build/styles/react-mdn.593cac165b36.css" rel="stylesheet" type="text/css" />
<link href="/static/build/styles/subscriptions.778c7f22cc86.css" rel="stylesheet" type="text/css" />
<link href="/static/build/styles/prism.a6f275e5032b.css" rel="stylesheet" type="text/css" />

<script async src='https://www.google-analytics.com/analytics.js'></script>
<script src="https://cdn.speedcurve.com/js/lux.js?id=108906238" async defer crossorigin="anonymous"></script>
<script async type="text/javascript" src="/static/build/js/perf.654b849a6fd9.js" charset="utf-8"></script>

<link href="/static/build/styles/mdn-subscriptions.bf637faaf5b0.css" rel="stylesheet" type="text/css" />

<script defer type="text/javascript" src="/static/build/js/react-main.73cb22117c96.js" charset="utf-8"></script>
<script defer type="text/javascript" src="/static/build/js/mathml.3cb4c04c0706.js" charset="utf-8"></script>
<script defer type="text/javascript" src="/static/build/js/auth-modal.31f8b35e40d7.js" charset="utf-8"></script>
<script defer type="text/javascript" src="/static/build/js/react-bcd-signal.0e7aead768ad.js" charset="utf-8"></script>
```

链接的资源主要分为三种:

- 样式表文件
- defer 脚本
- async 脚本

而下面的图片是 DCL 触发在性能面板中的截图, 右下角的蓝色方块表示的就是 DCL 触发的时间:

![image-20200404005650236](./assets\image-20200404005650236.png)

图片中左上角红色区域中的蓝色区域, 实际上是浏览器解析 CSS 所花费的过程, 而绿色区域中的黄色区域, 实际上是浏览器解析脚本和执行脚本的过程.

也就是说在 DCL 触发前, 浏览器是会处理 CSS 和 JavaScript 的, 会处理哪些内容跟呢?

- 符合媒体查询的样式
- 使用了 defer 的脚本

而图片中所展示的区域实际上无一例外都符合上述两点要求. 也就是代码中引入的资源中:

- 所有的样式
- 使用了 defer 的脚本

在完成上述内容解析后才会触发 DCL 事件.

现在我们再来看看 onload 事件, 根据 MDN 的解释 onload 事件是在整个页面被加载包括所有的依赖资源例如图片和样式表都被加载后触发. 在本例中甚至内嵌的 `<iframe>` 以及 `<iframe>` 所依赖的资源也同样被加载完成后触发的 onload 事件.

![image-20200404012034316](./assets\image-20200404012034316.png)

在这个图片中, 绿色区域内的蓝色方块代表的是 `<iframe>` 嵌套页面的资源下载过程, 而内部的紫色区域则是该 `<iframe>` 对应的样式. 随后 onload 事件触发.

总的来讲, onload 事件会在页面中所有的资源加载完成后触发, 但是不包括 XHR 或者动态载入的资源.

还有一个例外就是:

```html
<link rel="shortcut icon" href="/static/img/favicon32.7f3da72dcea1.png">
```

即浏览器标签页上的对应页面的标志, 在我的多次测试中都是 onload 事件完成后这个图标的资源的请求才发送. 也就是说这个图标是不会影响 onload 事件的.

# 参考

> https://calendar.perfplanet.com/2012/deciphering-the-critical-rendering-path/
>
> https://developer.mozilla.org/en-US/docs/Web/API/Window/load_event