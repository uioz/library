# 前言


有关 SPA 项目的 SEO 解决方案其实不多, 常见基本分为两种:

1. 改为 SSR 渲染
2. 使用预渲染

前者非常稳定但是对于已有的 SPA 项目进行改造与重写一个没有太大区别，后者只能对于那些"无论是那种用户访问返回的结果都一样"的页面适合。

总的来说都是十分的麻烦，不过如果你尝试搜索解决方案实际上智慧的网友已经给出了答案。

》 图片

这些方案的基本原理就是，使用代理服务器区分搜索引擎的爬虫和普通用户从而实现针对性的响应，普通用户响应原有的 SPA 项目也就是“纯粹的 index.html 页面”，而对于爬虫响应对应路由下的渲染好的HTML页面。

既然各位智慧无穷的网友开出了药方，看来我们需要手动熬制了。不过先慢着看看 github 有什么中成药没有:

》图片

没错实际上已经有了现成的解决方案。

我们看看它可以提供哪些优势:

1. 无需修改原有项目
2. 无需修改构建配置
3. 支持任意页面的渲染
4. 不受限于前端框架

我花了不到半个小时就配置好了, 足以证明它的易用性(虽然其他环境配置了半天)。

# Rendora 简介

**Rendora** 本身是一个代理服务器使用 GO 语言编写，支持多种配置以及对外提供了接口。

它的基本原理就是 Web页面的请求经过 **Rendora** 它会根据请求头 `user-agent` 来判断请求是爬虫还是普通用户, 普通用户直接代理到原有的Web服务器, 而爬虫的请求会经过无头浏览器(head-less browser) 处理生成一张在页面请求完成的 HTML 文件，然后在响应给爬虫。

明白了基本原理后我们不难想到只要是异步加载数据渲染然后再渲染内容的页面都适用。而且无论是爬虫还是普通用户他们实际上获取到的内容都是一样的。

# 安装与部署

官方文档中已经给出了安装方式，我就在这里直接照搬了，不过 Rendora 本身由 GO 语言编写，而且依赖了无头浏览器(实际使用的是 Chromium)还是有许多小坑要踩的。

我使用的系统是 Ubuntu18.04桌面版 ，但是其他的系统用户包括 windows 和 macos 都是可以的，安装 Rendora 方式稍有不同但是基本概念都是一致的。

## 安装 Golang

这里唯一的要求就是需要 Golang 版本在1.11及以上。

具体安装的方式请查看 Golang 的官方指南。

## 安装 Chromium

没错就是Chromium浏览器，我们可以通过命令行来控制Chromium浏览器以无界面的形式运行这也是“无头”一词的由来。同样的使用 `chrome` 浏览器也是可以的。

## 安装 Rendora

[项目地址](https://github.com/rendora/rendora)

安装方式:

```bash
git clone https://github.com/rendora/rendora
cd rendora
make build
sudo make install
```

另外你还可使用使用 docker:

```bash
docker run --net=host -v ./CONFIG_FILE.yaml:/etc/rendora/config.yaml rendora/rendora
```

**注意**:构建过程中会访问网络, 其中有些地址无法在国内访问这会导致构建失败, 国内用户没有开启代理的可以尝试在构建执行如下两条命令:

```bash
# 启动 go modules 特性
export GO111MODULE=on
# 设置 GOPROXY 为环境变量
export GOPROXY=https://goproxy.io
```

同样的其他平台的可以参考 [goproxy.io](https://goproxy.io/) 的官方指导.

## 编写配置文件





> http://www.runtester.com/detail/blog/20

> https://github.com/rendora/rendora

> https://github.com/rendora/rendora/issues/14