# 前言

[verdaccio](https://verdaccio.org/docs/en/configuration) 是一款开源用于创建私有 `registry` 的工具, 简单来说你可以用它来搭建一个自己的 `npm` 仓库, 可以实现绝大部分 `npm` 所提供的能力.

[verdaccio](https://verdaccio.org/docs/en/configuration) 基于 `node` 开发, 对于前端开发人员来说获取 [verdaccio](https://verdaccio.org/docs/en/configuration) 这款工具就像喝水一样简单.

工具的诞生往往是根据需求而出现的, 当你想要分享你的模块时候, 你可能会有如下的想法.

- 我希望我的内部模块不会对外公开
  - [verdaccio](https://verdaccio.org/docs/en/configuration) 就是为局部托管而生, 这是它的本职工作
- 我像要快速, 安全的模块托管体验
  - 在内网部署的 [verdaccio](https://verdaccio.org/docs/en/configuration) 毫无疑问很安全, 而且他还会自动缓存模块, 相当于给全局加了一层缓存然后在于内网的速度结合 absolutely
- 我想组织团队间的代码
  - 通过 `npm cli` 的提供的 scoped 功能或者通过 [verdaccio](https://verdaccio.org/docs/en/configuration) 的权限控制你都可以轻松的协调和组织不同项目间的模块
- 我想对模块添加权限访问控制
  - 当然没有问题, 你可以通过模块名称分配权限, 通过用户名称分配权限, 或者给每一个用户单独分配权限.

最后 [verdaccio](https://verdaccio.org/docs/en/configuration) 还会提供一个 GUI 虽然功能没有像 `npmjs.org` 那样强大, 但是提供了最最实用的功能.

![](C:\Users\zhao\Documents\library\article\assets\52916111-fa4ba980-32db-11e9-8a64-f4e06eb920b3.png)

# 配置 [verdaccio](https://verdaccio.org/docs/en/configuration) 

使用 `npm` 或者 `yarn` 等工具直接安装就是了:

```
npm install -g verdaccio
// 或者对于 yarn
yarn global add verdaccio
```

如果你没有特殊需求, 直接通过全局安装. 想要了解安装前置要求, 或者其他安装方式? 为了本篇文章的简洁请移步[官方文档](https://verdaccio.org/docs/en/installation#prerequisites)

## 启动 [verdaccio](https://verdaccio.org/docs/en/configuration) 

不需要任何启动配置, 直接在命令行中输入:

```
verdaccio
```

🎇你已经有了自己的私有仓库, 怎样是不是很简单.

当然不传入和任何参数的情况下 `verdaccio` 会使用默认的配置, 当然我们也可以手动的为其指定配置文件.

你可以直接打开浏览器前往默认的 `locahost:4873` 体验 GUI 或者继续后面的教程, 所以如果你的 4873 端口被占用也是不要紧的.

## 配置文件路径

知道了配置文件的路径, 心中才会踏实, 最简单的方式就是在执行 `verdaccio` 命令后查看终端的首行输出:

![image-20210222231042100](./assets\image-20210222231042100.png)

第一行就是配置文件的地址.

[官方配置文件地址文档](https://verdaccio.org/docs/en/cli#default-config-file-location)

# 第一个私有包

上传一个模块总共分为 3 步骤:

1. 搞到一个 `npm` 模块
2. 登入到要上传的 registry
3. 在项目下执行 `npm publish`

## 创建一个 npm 模块

随便打开一个目录然后执行, 让这个目录变为一个 npm 模块:

```
npm init -y
```

![image-20210224224014170](./assets\image-20210224224014170.png)

然后添加一个 `index.js` , 内容随意例如:

```javascript
export function echo () {
    console.log('hello world!');
}
```

## 登入到 registry

registry 就是模块集中注册的地方, `npm cli` 默认的注册处是 `https://registry.npmjs.org/` , 如果你在国内你的地址很可能已经换成了淘宝源那么这个地址会有所差别.

可以执行下面的命令来可以观察默认 `npm cli` 的配置:

```
npm config list
```

![image-20210224224819991](./assets\image-20210224224819991.png)

我们所要做的是设置 registry 为我们本地的 `verdaccio` 服务:

```
npm set registry http://localhost:4873/
```

很多人对于符合 npm 规则的 registry 的工作方式不太熟悉, 这里简单的说明一下, 想要推送一个模块除了指定一个你想推送的 registry, 还需要通过 `npm cli` 工具借助于交互式命令登陆到这个 registry, 然后才可以推送.

对于默认的 `https://npmjs.org/` 你需要在该网站上注册一个账号, 然后再在本地登陆, 对于 `verdaccio` 你不需要注册, 在你通过命令行登陆的时候如果这个用户不存在则直接创建一个新的账户.

现在我们通过交互式命令添加一个新的账户:

```
npm adduser
```

![image-20210224231430747](./assets\image-20210224231430747.png)

## 推送模块到 registry

好了执行下面的代码来进行推送吧:

```
npm publish
```

![image-20210224231756827](./assets\image-20210224231756827.png)

现在前往 `http://localhost:4873/` 你的首个模块已经推送完成啦!

![image-20210224231948137](C:\Users\zhao\Documents\library\article\assets\image-20210224231948137.png)

# 第一个作用域模块

## 作用域模块简介

作用域模块(scoped module) 简单来将就是命名空间, 对于 `https://npmjs.org/` 上的每一个用户或者组织它们的账户/组织名都是唯一的, 将它们的账户作为命名空间这样一来这个账户下的模块名称就可以现有的公开模块重名.

例如我有一个 `example` 的账户, 我想推送一个我自己修改后的 `react` 模块, 显然这和现有的模块重名了, 因为已经存在一个公开的 `react` 模块, 那么我就可以将他设置为作用域模块, 名称就成为了 `@example/react`. 是不是有点熟悉? `@vue/cli` 和 `@bable/core` 就是作用域模块.

另外如果你使用过 `docker` 用于镜像托管的 `https://hub.docker.com/` 上的镜像名称与 npm 的作用域模块名称有着相似的设计.

## 作用域模块的特点

1. 不同作用域下的模块可以重名
2. 可以为 registry 配置指定的作用域
3. 模块名称格式如下 `@作用域/模块名称`

如果我们存在多个团队, 不同团队内部拥有自己的私有 npm 托管, 如果需要模块互相借用, 可能会出现模块名称冲突的问题. 利用作用域模块我们可以将修改模块命名为 `@团队名称/模块名称` 或者 `@项目名称/模块名称`.

另外将 `registry` 与作用域模块关联后对于公开模块的下载, 流量就不会经过 `verdaccio` 当然这是不需要使用 `verdaccio` 缓存的情况下, 这些内容我们后面的章节会提到.

## 两种设置作用域的方式

有两种主要的设置作用域模块的途径:

1. 给现有的模块添加作用域

```
npm init --scope <scope name>
```

2. 登陆 registry 并为其配置作用域和地址

```
npm adduser --registry=<registry> --scope=@<scope name>
```

当下载和推送模块的时候只要作用域匹配, 那么会下载和推送到该 `registry` 上去.

实践一下, 我们现在重置 npm 到初始状态:

```
// 重置默认 registry
npm set registry https://registry.npmjs.org/

// 取消登陆状态
npm logout
```

进入到项目目录通过为其创建一个 `example` 作用域:

```
// 该命令会重写当前的 package.json, 同时也是交互式命令, 一路回车即可
npm init --scope example
```

然后登陆到 `http://localhost:4873` 同时指定作用域:

```
// 第二次登陆如果用的是之前的账号, 密码则必须和之前一样
npm adduser --registry=http://localhost:4873 --scope=example
```

然后推送模块:

```
npm publish
```

在继续之前这里涉猎一些一些 `npm` 与 [verdaccio](https://verdaccio.org/docs/en/configuration) 在包访问控制上的一些区别, 一个作用域模块默认不会被公开基本等同于作者可见, 也就是 `npm` 在模块上添加了访问控制.

但是 [verdaccio](https://verdaccio.org/docs/en/configuration) 去掉了有关访问控制的符合 `npm` 的实现, 基于 `npm access` 的命令无法使用, 配置模块访问则需要修改配置文件.

如果你希望你的 `npm` 模块可见在推送时候可以指定如下的命令:

```
npm publish --access=public
```

如果这个模块已经被提交则需要使用来修改:

```
npm access public
```

> https://docs.npmjs.com/cli/v7/commands/npm-access#details

回到 [verdaccio](https://verdaccio.org/docs/en/configuration) 这边, 默认的配置中任何人都可以访问作用域模块, 所以进入到 GUI 你就可以看到这个模块了:

![image-20210228182100438](./assets\image-20210228182100438.png)

配置文件的使用方式在后面的章节中会介绍.

好了通过下列命令就可以从我们指定的 `registry` 上安装我们的作用域模块:

```
npm install @example/verdaccio-example
```

试想一下如果存在多个开发项目组, 如果所有的模块都基于 `@项目名称/模块` 名称的形式, 不同项目间的引用模块将会轻松的多.

# 配置文件入门

## 简介

之前我们已经提交如何寻找默认配置文件的位置, 这里不在赘述.

[官方文档-配置文件](https://verdaccio.org/docs/en/configuration)

`verdaccio` 的配置文件并不复杂, 本章节只会挑重点介绍, 否则就成为翻译文档了, 所以不要忘记去文档中浏览其他的功能配置.

# 设置模块

默认配置文件的包访问控制如下:

```yaml
packages:
  # 可以使用通配符来匹配包名, 这个条件匹配的是作用域模块
  '@*/*':
    # 访问权限, 基本等用于是否可以被搜索到
    access: $all
    # 托送权限
    publish: $authenticated
    # 代理配置, 在模块缓存一节中会介绍
    proxy: npmjs
  # 这个条件匹配所有模块
  '**':
    proxy: npmjs
```

除了 `access` `publish` 和 `proxy` 还有一个 `storage` 选项用于控制匹配模块的相对于存储目录的路径.

说实话我对这些权限组概念无法理解, 为了不带来错误的信息, 这里只给出具体的用法, 想要深入了解的可以去看官方文档.

可用的内置权限组如下:

```
'$all', '$anonymous', '@all', '@anonymous', 'all', 'undefined', 'anonymous', $authenticated
```

- $all 表示全部用户
- $anonymous 排除登陆用户
- $authenticated 表示登陆用户

已注册用户名可以用于权限控制, 通过这种方式可以进行更加精细的操作:

```yaml
packages:
  'npmuser-*':
    access: npmuser
    publish: npmuser
```

上面这份配置只要符合 `npmuser-*` 规则的模块, 只有账号为 `npmuser` 的用户才拥有权限.

# 用户控制

## 设置多个用户组

```yaml
  'company-*':
    access: admin internal
    publish: admin
    proxy: server1
  'supersecret-*':
    access: secret super-secret-area ultra-secret-area
    publish: secret ultra-secret-area
    proxy: server1
```

## 禁用一组模块

```yaml
packages:
  'old-*':
  # 不定义 access 和 publish 即可
  '**':
    access: $all
    publish: $authenticated
```

## 禁止模块代理到其他 registry

**注意**: `proxy`  的用法在下一节描述

```yaml
packages:
  'jquery':
    access: $all
    publish: $all
    unpublish: root
  'my-company-*':
    access: $all
    publish: $authenticated
    unpublish:
  '@my-local-scope/*':
    access: $all
    publish: $authenticated
    # unpublish: property commented out
  '**':
    access: $all
    publish: $authenticated
    proxy: npmjs
```

这份配置有如下的目的:

1. 允许任何人读取和托送保存在本地的 `jquery` 模块, 但是只有 `root` 用户才可以移除它
2. `my-company-*` 模块只允许所有人访问, 只有登陆后的才可以推送, 但是不允许移除这个模块
3. `@my-local-scope/*` 模块允许所有人访问, 只有登陆后的人才可以推送, 推送的人有权限移除它
4. 其他的模块允许所有人访问, 只有登陆后才可以推送

## 批量创建账号密码

我自己并没有尝试批量创建用户, 但是这是可行的. 默认的访问控制基于 [htpasswd](https://github.com/verdaccio/monorepo/tree/9.x/plugins/htpasswd) 注册的用户被保存到了 `htpasswd` 文件中. 这个文件和配置文件保存在同一个目录中.

打开该文件后你可以看到如下的文本, 这些是我通过 `verdaccio` 登陆后所创建的, 基本上就是 用户名 + 密码哈希 +    创建类型 + 日期, 其中只有账号和密码哈希是必要的.

```
admin:Veu47j6NfSujQ:autocreated 2021-02-05T08:08:22.684Z
example:x01Ba3hJe4d2c:autocreated 2021-02-24T15:13:27.132Z
teama:GV.PRNbm5rgi6:autocreated 2021-02-24T15:59:20.618Z
```

`htpasswd` 仓库简单的介绍了这个文件:

> The htpasswd file contains rows corresponding to a pair of username and password separated with a colon character. The password is encrypted using the UNIX system's crypt method and may use MD5 or SHA1.

另外你可以用这个网站(需翻墙)来在线创建.

> https://hostingcanada.org/htpasswd-generator/

但是想要批量创建可能需要写个脚本.

# 模块代理与缓存

## 模块代理

如果全局 `registry` 设置为了 `verdaccio` 如果我们下载一个不存在于 `verdaccio` 上的模块, `verdaccio` 会读取上一级的 `registry` 默认就是 `https://registry.npmjs.org/`.

在配置文件中我们可以通过 `uplinks` 声明多个上游:

```yaml
uplinks:
  npmjs:
   url: https://registry.npmjs.org/
  server2:
    url: http://mirror.local.net/
    timeout: 100ms
  server3:
    url: http://mirror2.local.net:9000/
  uplink1:
    url: http://localhost:55666/
```

声明好的上游用于 `packages.proxy` 选项:

```yaml
packages:
  '@scope/*':
    access: $all
    publish: $all
    proxy: server2
  'private-*':
    access: $all
    publish: $all
    proxy: uplink1
  '**':
    access: $all
    publish: $all
    proxy: npmjs
```

上面这份配置中如果符合模块名称匹配规则的模块不存在本地服务上对于:

- `@scope/*` 会向 `server2` 也就是 `http://mirror.local.net/` 查询
- `private-*` 会向 `uplink1` 也就是 `http://localhost:55666/` 查询
- 其他的模块则会向 `npmjs` 也就是 `https://registry.npmjs.org/` 查询

当然默认的 `npmjs` 可以指向 `https://registry.npm.taobao.org` 对于国内的网络环境更加友好.

## 模块缓存

`verdaccio` 默认会缓存上游下载下的模块, 这是十分有意义的, 一般来说重复下载同一个模块的几率要比下载一个新的模块几率高得多.

有关缓存的配置项目不多, 主要集中在 `uplinks` 选项上:

```
uplinks:
  npmjs:
   # 表示是否开启缓存
   cache:true
   # 表示缓存多久后失效
   maxage:10m
   url: https://registry.npmjs.org/
```

> 官方文档地址 https://verdaccio.org/docs/en/what-is-verdaccio