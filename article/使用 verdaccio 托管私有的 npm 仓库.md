# 前言

[verdaccio](https://verdaccio.org/docs/en/configuration) 是一款开源用于私有 `npm` 仓库的工具, 简单来说你可以用它来搭建一个自己的 `npm` 仓库, 可以实现所  http://npmjs.org/ 提供的功能.

[verdaccio](https://verdaccio.org/docs/en/configuration) 基于 `node` 对于前端开发人员来说获取 [verdaccio](https://verdaccio.org/docs/en/configuration) 这款工具就像喝水一样简单.

工具的诞生往往是根据需求而出现的, 那么? 为什么需要一个私有的 `npm`  仓库呢? 好吧, 当你想要分享你的模块时候, 你可能会有如下的想法.

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

知道了配置文件的路径, 心中才会踏实, 最简单的方式就是在执行 `verdaccio` 命令后查看终端的输出:

![image-20210222231042100](./assets\image-20210222231042100.png)

看到了吗, 第一行就是配置文件的地址.

[官方文档地址请点击](https://verdaccio.org/docs/en/cli#default-config-file-location)

# 第一个私有包

上传一个模块总共分为 3 步骤:

1. 搞到一个 `npm` 模块
2. 登入到要上传的 registry
3. 在项目下执行 `npm publish`

## 创建一个 npm 模块

## 登入到 registry

## 推送模块到 registry

# 第一个作用域模块

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

