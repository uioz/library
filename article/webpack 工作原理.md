# webpack 工作原理

`webpack` 和 `webpack-cli` 以及 `webpack-dev-server` 是三种不同的东西, 在实际使用过程中这三者一般都是同时使用这造成了一些混淆很少有人了解它们之间的边界, 上述三个内容对于 "打包" 这个关键功能来说只有 `webpack` 这个模块是需要的, 而 `webpack-cli` 和 `webpack-dev-server` 可有可无.

为什么这么说, 你很有可能没有真正的直接的使用过 `webpack` 模块在 `Node.js` 中, 这是一个简单的示例:

```javascript
const webpack = require('webpack');
const webpackConfig = require('./webpack.config.json');
const compiler = webpack(webpackConfig);
```

利用 `compiler` 实例提供了编译方法调用后可以进行编译, 甚至监视文件的改变后在进行编译等等其他功能, 总之 `compiler` 就是 `webpack` 的具体实例.

不过你也猜出来 了 `webpackConfig` 就是平常我们使用的 `webpack` 的配置文件, 那么 `webpack-cli` 担任的就是读取配置以及读取命令行中的参数来进行合并生成一个配置文件, 然后就和例子中一样执行 `new webpack(webpackConfig)` 仅此而已.

现在让我们开始正式的关注 `webpack` 本身. 非常值得提前熟悉的概念就是: 在 `webpack` 内部实际上所有的内容都是由插件构成的, 在每一个配置选项中背后都是插件在处理, 而 `webpack` 通过 `Tapable` 这个模块来当作 `webpack` 的内核. 这个模块的工作类似于 `EventEmitter` 但是更为强大.

`webpack` 的工作流程实际上并不复杂, 取决于观察的方式 `webpack` 的主要工作流程大约为 6~7 步:

1. 初始化配置, 从配置文件和 Shell 语句中读取与合并参数，得出最终的参数(实际是由 `webpack-cli` 完成的)
2. 利用配置来创建一个 `Compiler`
3. 调用插件的 apply 方法, apply 方法主要用于做一些初始化工作(插件是一个 "类" 需要提供 apply 方法)
4. 编译模块, 从入口文件出发, 递归的解析依赖
   1. 对每一个文件调用对应的 loader
   2. 通过 loader 处理后的结果转为 AST 进行后续处理
5. 所有模块及其依赖的模块都通过Loader转换完成后，根据依赖关系开始生成Chunk
6. 输出 chunk 到文件系统

如果 `webpack-cli` 使用了 `watch` 选项, 每当文件修改后 `webpack` 就会从第四步重新走一遍流程, 当然没有发生修改的文件 `webpack` 不会进行重新编译.

# 理解代码分割

`webpack` 中 `entry` 可以是一个也可以是多个, 简单来讲一个入口对应一个独立的 chunk, 而多个入口对应多个 chunk.

为什么, 因为编译流程是根据入口来构建依赖的, 所以不同的入口依赖不同.

那么 a.js 和 b.js 引用了 c.js , 被依赖两次的 c.js 会被重复打包, 解决这种问题就是 "代码分割" 的职责.

`webpack` 提供了两种方式去解决这种问题, 第一种非常简单在入口处显示指定共同的依赖:

```javascript
  module.exports = {
    mode: 'development',
    entry: {
     index: { import: './src/index.js', dependOn: 'shared' },
     another: { import: './src/another-module.js', dependOn: 'shared' },
     shared: 'lodash',
    },
    output: {
      filename: '[name].bundle.js',
      path: path.resolve(__dirname, 'dist'),
    },
  };
```

在这个例子中 `lodash` 被 `index` 和 `another` 依赖(或者叫做分享), 这样一来在最终生成的代码中 `index` 和 `another` 引用的 `lodash` 是同一份.

第二种也是最常用的一种, 基于 `SplitChunksPlugin` 插件:

```javascript
  module.exports = {
    mode: 'development',
    entry: {
      index: './src/index.js',
      another: './src/another-module.js',
    },
    output: {
      filename: '[name].bundle.js',
      path: path.resolve(__dirname, 'dist'),
    },
   optimization: {
     splitChunks: {
       chunks: 'all',
     },
   },
  };
```

# 参考

> https://www.jianshu.com/p/53f370cc8c59
>
> https://segmentfault.com/a/1190000015088834?utm_source=tag-newest#item-3
>
> https://www.imweb.io/topic/59324940b9b65af940bf58ae
>
> https://raw.githubusercontent.com/sokra/slides/master/data/how-webpack-works.pdf
>
> https://youtu.be/xse6JKcfbzs