# 前言

本文章写于 2019-07-05 请注意时效性。

有关 SPA 项目的 SEO 友好的解决方案其实不多, 常见的解决手段如下:

1. 将 SPA 项目改为 SSR 渲染
2. 使用预渲染

前者非常稳定但是对于已有的 SPA 项目进行改造与重写一个没有太大区别而且需要注意的问题有很多而且耗时长，后者只能对于那些"无论是哪种用户访问返回的结果都一样"的页面合适十分被动。

总的来说都是十分的繁琐，不过依然有可以避开修改原有代码的解决方案, 例如下面的这个：

> https://www.cnblogs.com/lipten/p/9609678.html

这些方案的基本原理就是，使用代理服务器区分搜索引擎的爬虫和普通用户从而实现针对性的响应，普通用户响应原有的 SPA 项目也就是“纯粹的 index.html 页面”，而对于爬虫响应对应路由下的渲染好的HTML页面。

![2019-07-05 22-26-06](/home/zhao/文档/library/article/assets/2019-07-05 22-26-06.png)

既然各位智慧无穷的网友开出了药方，看来我们需要手动熬制了。不过先慢着看看 github 有什么现成的药没有:

![2019-07-05 22-15-43](/home/zhao/文档/library/article/assets/2019-07-05 22-15-43.png)

没错实际上已经有了现成的解决方案。

# Rendora 简介

**Rendora** 本身是一个代理服务器使用 GO 语言编写，支持配置文件以及对外接口。

使用 Rendora 可以相较于其他方案有如下的优势:

1. 无需修改原有项目
2. 无需修改构建配置
3. 支持任意路由页面的渲染
4. 不受限于前端框架与所使用到的技术
5. 搜索引擎爬虫和普通用户获取到的数据一致

它的基本原理就是Web页面的请求经过 **Rendora** 的它会根据请求头 `user-agent` 来判断请求是爬虫还是普通用户, 普通用户直接代理到原有的Web服务器, 而爬虫的请求会经过无头浏览器(head-less browser) 处理生成一张在页面请求完成的 HTML 文件，然后在返回给爬虫。

![2019-07-05 22-33-57](/home/zhao/图片/2019-07-05 22-33-57.png)

明白了基本原理后我们不难想到只要是异步加载数据然后再利用数据渲染内容的页面都适用。而且爬虫和普通用户两者最终获取到的数据可以高度一致。

# 安装

[Rendora](https://github.com/rendora/rendora) 官方文档中已经给出了安装方式，我就在这里直接照搬了，不过 Rendora 本身是由 GO 语言编写，而且依赖了无头浏览器(我实际使用的是 Chromium)还是有许多小坑要踩的。

在本文中我使用的系统是 Ubuntu18.04桌面版 ，但是其他的系统用户 windows 和 macos 都是可以安装以及使用的，安装 Rendora 方式稍有不同但是基本概念都是一致的。

## 基本依赖

1. Golang 1.11及以上
2. chromium 浏览器或者 google-chrome 浏览器，要确保可以在环境变量中可以访问到他们

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

# 编写配置文件

Rendora 是基于配置文件运行的, 在运行前我们需要熟悉一下配置文件.

[配置手册](https://rendora.co/docs/)

配置文件支持多种格式, 这里我就使用 Web 端最常见的 JSON 格式, 需要注意 Rendora 不会检查拼写错误, 请多多复制.

默认情况下我们只需要指定2个参数就可以了:

```json
{
    "backend": {
        "url": "http://127.0.0.1:8000"
    },
    "target": {
        "url": "http://127.0.0.1"
    }
}
```

| 参数    | 含义                     |
| ------- | ------------------------ |
| backend | 原来向用户提供服务的地址 |
| target  | 无头浏览器请求的地址     |

请注意: 因为 Rendora 本质上是一个代理服务器也会启动端口监听(默认3001端口), 这两个参数具体填写的内容取决于后端的配置, 例如一个常见的配置可能是下面这个样子:

```
nginx->Rendora->App Server
```

但也有可能是反过来的:

```
Rendora->nginx->App Server
```

例如: 我在本地的服务器上监听了80端口用于托管项目的静态文件, 那么实际上这两个参数的配置是一样的, 因为原有的地址和浏览器请求的地址是一致的.

此外为了避免和本地的端口冲突这里还有两个值是需要注意的:

```
// TODO head-less 端口
// TODO 本地监听端口
```



# 运行

> http://www.runtester.com/detail/blog/20

> https://github.com/rendora/rendora

> https://github.com/rendora/rendora/issues/14