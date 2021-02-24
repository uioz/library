# 前言

[verdaccio](https://verdaccio.org/docs/en/configuration) 是一款开源用于私有 `npm` 仓库的工具, 简单来说你可以用它来搭建一个自己的 `npm` 仓库, 可以实现所  `https://npmjs.org/` 提供的功能.

[verdaccio](https://verdaccio.org/docs/en/configuration) 基于 `node` 对于前端开发人员来说获取 [verdaccio](https://verdaccio.org/docs/en/configuration) 这款工具就像喝水一样简单.

工具的诞生往往是根据需求而出现的, so 你为什么需要一个私有的 `npm`  仓库呢? 好吧, 当你想要分享你的模块时候, 你可能会有如下的想法.

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

如果你没有特殊需求, 直接通过全局安装. 想要了解安装前置要求? 为了本篇文章的简洁请移步[官方文档](https://verdaccio.org/docs/en/installation#prerequisites)

## 启动 [verdaccio](https://verdaccio.org/docs/en/configuration) 

不需要任何启动配置, 直接在命令行中输入:

```
verdaccio
```

🎇你已经有了自己的私有仓库, 怎样是不是很简单.

![image-20210222231042100](./assets\image-20210222231042100.png)

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

### 作用域模块简介

作用域模块(scoped module) 简单来将就是命名空间, 对于 `https://npmjs.org/` 上的每一个用户或者组织它们的账户/组织名都是唯一的, 将它们的账户作为命名空间这样一来这个账户下的模块名称就可以现有的公开模块重名.

例如我有一个 `example` 的账户, 我想推送一个我自己修改后的 `react` 模块, 显然这和现有的模块重名了, 因为已经存在一个公开的 `react` 模块, 那么我就可以将他设置为作用域模块, 名称就成为了 `@example/react`. 是不是有点熟悉? `@vue/cli` 和 `@bable/core` 就是作用域模块.

另外如果你使用过 `docker` 用于镜像托管的 `https://hub.docker.com/` 上的镜像名称与 npm 的作用域模块名称有着相似的设计.

### 作用域模块优势

作用域模块有两个最大的优势:

1. 作用域下的模块可以重名
2. 不同的作用域可以指定不同的 registry

如果我们的开发团队足够多, 总是会构成复杂的网络拓扑, 提前设置好的作用域就会在此时发挥作用:

TODO: 修改配图 优势区分当前模块和其他 registry 模块

当然不使用作用域模块的情况下也存在其他的解决方式, 而且这种原生的方式还需要 `teamC` 持有 `teamA` 和 `teamB` 的账号, 在 `verdaccio` 默认配置的情况下不会验证用户, 如果严格限制随意注册就成为了问题.

## 两种设置作用域的方式

1. 给现有的模块添加作用域

```
npm init --scope <scope name>
```

2. 登陆用户到指定的 registry 且配置作用域

```
npm adduser --registry=<registry> --scope=@<scope name>
```

这样该 `registry` 下的模块就会被添加了一个作用域, 不会和当前的 registry 冲突.

# 配置文件入门

# 包访问控制

# 用户控制

# 模块缓存

# npm 拓展

对于哪些没有使用过 `npm` 进阶功能的人来说下面的几个概念是很实用的.

## registry

## 登入/登出

## scoped(作用域)

## team

