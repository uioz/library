# 前言

:warning:本篇文章建议亲自动手尝试.

最近研究了 webpack 运行时源码, 在这篇文章中记录了我的探索 webpack 这个复杂的玩具方式, 并且以图形的形式将 webpack 运行时的流程给记录的下来. 

## 我们讨论的是什么

这篇文章主要记录的是 webpack 在将文件打包后是如何在浏览器中加载以及解析的流程.

## 手段

webpack 实在是太复杂的了, 为了避免额外的干扰, 我使用最小化实例的方式来进行探究 webpack 是如何加载和解析模块文件的. 

具体的手段:首先准备几个非常简单的 js 文件, 其次准备其对应的 webpack 配置文件, 逐一测试后观察其输出, 阅读其源码, 然后得出结论.

## 简单总结

webpack 的运行时在解析同步模块的时候就和 nodejs 规则如出一辙.  

什么意思, 每一个模块在运行前都会被一个函数进行包裹:
```javascript
(function (module, exports, __webpack_require__) {
    // do something
})
```
看起来和 nodejs 一样, 作用起来也一致, 这里的当前模块的导出模块和导入模块就是:
- exports 
- \__webpack_require__

而当一个模块被需要的时候, webpack 就会执行这个函数获取其**对应的 exports对象.**

**注意**:我们需要的是模块执行完成后的 exports 对象而不是这个模块函数.  

在 webpack 中我们可以使用 ESM 规范和 commonjs 规范来加载模块, 无论使用哪种规范, 实际上都可以被这种方式来包裹起来, 因为这两种常见的方式中都存在着相同的导入导出的概念, 所以 webpack 将这两种形式进行了统一包装消除了差异. 

**对于异步模块**这里有两种情况都是属于异步的情况:
- 文件的加载是无顺序的, 入口文件有可能是最后才被加载
- 使用 `import()` 语法, 引入的模块.

现在你需要知道的是, webpack 运行时完全不依赖文件加载顺序, 无论文件加载顺序是何种方式, webpack 都可以轻松应对. 

`import()` 语法常用于代码切割或者叫做懒加载, 这种情况下 webpack 使用script引入打包后的文件, 然后使用Promise语法来进行异步处理(后续会有进一步的讨论). 

## 依赖

```
"webpack": "^4.31.0",
"webpack-cli": "^3.3.2"
```

只需要 webpack 本身就可以了

## 建议

个人建议可以直接上手把玩一番, 文章中源码不多都是解释性质的内容, 只有当自己试过了以后才可以理解透彻. 

# 实践

## 同步模块-不分离runtime

提供一个 `index.js` 内部就一个`console.log('hello world')`, 然后进行打包来检测

**webpack.config.js**:

```javascript
{ // 省略了导出
    mode: 'development',
    entry: {
    	app: './src/index.js',
    },
    output: {
    	filename: '[name].js'
    },
    devtool:'hidden-source-map', // 这样做生成的代码中注释更加少一些, 不是为了sourceMap
}
```

webpack 默认情况下会输出一个 `app.js` 而且只会有100行(这还是在有没用的注释情况下), 打开文件后会发现一个IIFE函数, 这里包含两部分:

- runtime本身, 即 IIFE 函数体
- 模块内容, 及 IIFE 函数的参数

**注意**:实际上 IIFE 函数内部有些冗余代码, 这些冗余代码实际上是为特殊情况和异步情况准备的, 所以不用太担心看不懂, 某些内容结合后续更多分析就看起来非常简单了.

**在同步的引入中, runtime本体内部会显示的嵌入入口文件的模块id, 而当前的配置下 webpack  使用文件路径来作为 模块的唯一id.**

在IIFE函数执行到尾部的时候 webpack 会利用这个id作为起点来进行模块的解析和执行.  

IIFE函数的参数:

```javascript
{
    "./src/index.js": (function (module, exports) {
    	console.log('hello world')
    })
}
```

**图片**:执行流程分析

![parse-single-file](F:\library\article\assets\parse-single-file.jpg)

## 同步模块-不分离runtime-导入和执行

现在我们在单纯运行代码的模块 `index.js` 中添加一个导出,然后观察在打包后的文件中会有什么样的改变:

```javascript
{
  "./src/index.js": (function (module, __webpack_exports__, __webpack_require__) {
    "use strict";
    __webpack_require__.r(__webpack_exports__); __webpack_require__.d(__webpack_exports__, "echo", function () { return echo; });
    const echo = () => {
      console.log('hello world');
    }
    echo();
  })
}
```

果然打包后的内容只有IIFE函数的参数有变化, 我们定睛一看多出了两个函数调用, 这是什么鬼:

- `__webpack_require__.r` 用来给 `exports` 添加一个描述标识这个模块是ESM模块
- `__webpack_require__.d` 用于定义模块导出, 和对于直接向 `exports` 添加一个属性不同, 使用这个函数定义的属性都是不可变的.

## 同步模块-不分离runtime-多个文件

现在我们来在代码中添加导入和导出, 来测试一下 webpack 多个同步模块之间引用是何种情况.

我建立一个另外一个文件 `demo.js` 这个文件负责导出一个函数, 而原来的 `index.js` 导入这个函数后执行这个函数. 

构建后明显的变化就是在IIFE函数的参数中, 多了些内容这里多出去的内容就是新增了一个 `demo.js` 所引起的.

```javascript
{
  "./src/demo.js": (function (module, __webpack_exports__, __webpack_require__) {

    "use strict";
    __webpack_require__.r(__webpack_exports__); __webpack_require__.d(__webpack_exports__, "demo", function () { return demo; });
    const demo = () => {
      console.log('hello world')
    }
  }),
  "./src/index.js": (function (module, __webpack_exports__, __webpack_require__) {
    "use strict";
    __webpack_require__.r(__webpack_exports__);
    var _demo__WEBPACK_IMPORTED_MODULE_0__ = __webpack_require__(/*! ./demo */ "./src/demo.js");
    Object(_demo__WEBPACK_IMPORTED_MODULE_0__["demo"])();
  })
}
```

这里提示一下 webpack runtime 提供的不同函数的功能:

- `__webpack_require__ ` 字如其名用于引入其他模块
- `__webpack_require__.r` 用来给 `exports` 添加一个描述标识这个模块是ESM模块
- `__webpack_require__.d` 用于定义模块导出, 和对于直接向 `exports` 添加一个属性不同, 使用这个函数定义的属性都是不可变的.

那么在模块id为 `./src/index.js` 中使用了导入模块也就是 `__webpack_require__` 而在 `./src/demo.js` 中导出模块也就是 `__webpack_require__.d`.

**图片**:执行流程分析

![parse-export-single](F:\library\article\assets\parse-export-single.jpg)

## 异步模块-运行时分离

这里我们将运行时和 `demo.js` and `index.js` 进行分离, 这里相较于上一步我们需要修改一下配置文件:

```javascript
{
    mode: 'development',
    entry: {
    	app: './src/index.js',
    },
    output: {
    	filename: '[name].js'
    },
    devtool:'hidden-source-map', // 这样做生成的代码中注释更加少一些, 不是为了sourceMap
    optimization: {
        runtimeChunk: {
        	name: 'runtime' // 将runtime分离
        },
	}
}
```

这个时候runtime被分离为一个单独的文件, 而 `demo.js` 和 `index.js` 组成一个一个 chunk 叫做 `app.js`.

我们知道浏览器在加载一个文件的时候, 默认的情况下是解析 HTML 中所有的script标签中的内容后才执行的.

也就是说 `javascript` 在浏览器中的执行会受到 script 标签顺序的影响.

显然 `app.js` 的执行是依赖 `runtime.js` 的, 那么违反了加载顺序是否可以正常执行呢?

**答案**:可以 webpack 运行时完全作为最后一个文件加载, 换句话说就是不会受加载顺序的影响, 以及是否同步加载的影响.

此时输出的 `runtime.js` 代码向较于同步的版本, 多出了一半的代码, 这些代码就是用于处理多个文件直接加载处理的. 

长话短说之前的多文件同步加载的版本, 只有模块的概念(模块全局唯一). 当有多个文件的时候我们会将文件进行拆分为 chunk 此时模块就属于 chunk 中的内容.

```javascript
(window["webpackJsonp"] = window["webpackJsonp"] || []).push([
  ["app"], // chunk 名称
  { // 模块
    "./src/demo.js": (function (module, __webpack_exports__, __webpack_require__) {
      "use strict";
      __webpack_require__.r(__webpack_exports__);
      __webpack_require__.d(__webpack_exports__, "demo", function () { return demo; });
      const demo = () => {
        console.log('hello world')
      }
    }), "./src/index.js": (function (module, __webpack_exports__, __webpack_require__) {
      "use strict";
      __webpack_require__.r(__webpack_exports__);
      var _demo__WEBPACK_IMPORTED_MODULE_0__ = __webpack_require__(/*! ./demo */ "./src/demo.js");
      Object(_demo__WEBPACK_IMPORTED_MODULE_0__["demo"])();
    })
  },
  [["./src/index.js", "runtime"]] // 第二列表项后的内容是这个chunk所依赖的chunk
]);
```

那么 webpack 运行时是如何做到文件顺序加载乱掉也可以正常执行内容呢?

webpack 运行时内部维护了一个数组变量, 这个变量被挂载在 window 对象上:

```javascript
window["webpackJsonp"] = []
```

无论是 runtime 还是普通的 chunk 都会在IIFE函数中试图去读取这个属性, 如果没有读取到就为其赋值一个数组.

```javascript
(window["webpackJsonp"] = window["webpackJsonp"] || []).push(xxx) // 处了runtime,每一个chunk都有哦
```

runtime 如果读取到了数组, 也就意味着 runtime 加载之前有其他 chunk 加载了, 此时 runtime 只要读取这个数组中的内容然后在进行解析上之前加载完成的 chunk 就OK了(在实际操作中他会修改这个数组对象改变其行为,使得后续的 chunk 调用的实际上是 runtime 上的 chunk 解析函数).

**图片**:执行流程分析

![async-chunk-mulit-file](F:\library\article\assets\async-chunk-mulit-file.jpg)

## 异步模块-`import()`语法

`import()` 语法才是 webpack 中实打实的异步模块. 当你使用 `import()`的语法来懒加载模块的时候 runtime 又会提供一些包装, 没错我们的 runtime 中的代码又变多了. 

为了增加点难度我们多增加了一个文件, 不过这是最后一节了, 放心吧难度不会再次提高了:joy:.

- demo1.js 导出一个函数.
- demo.js 引入 `demo1.js` 中导出的函数并且再次导出.
- index.js 使用 `import()` 语法来引入 `demo.js` 中的导出内容.

```javascript
// demo1.js
export const echo = ()=>{
  console.log('hello world')
}
```

```javascript
// demo.js
export { echo } from "./demo1";
```

```javascript
// index.js
import(/* webpackChunkName: "demo" */'./demo').then(({echo})=>{
  echo();
});
```

我们来看一下 `app.js` 做了什么:

```javascript
(window["webpackJsonp"] = window["webpackJsonp"] || []).push([["app"], {
  "./src/index.js": (function (module, exports, __webpack_require__) {

    __webpack_require__.e(/*! import() | demo */ "demo")
      .then(__webpack_require__.bind(null, /*! ./demo */ "./src/demo.js"))
      .then(({ echo }) => {
        echo();
      })
      
  })

}, [["./src/index.js", "runtime"]]]);
```

这里的引入模块使用了  `__webpack_require__.e` 再次之前这个 API 是不存在的. 

而且它后面还连接了两个 then  第一个是获取依赖模块的导出, 第二个是用于依赖的执行.

`__webpack_require__.e` 主要完成了如下的事情:

1. `./index.js` 是入口模块 runtime 会首先执行对应的函数, 此时``__webpack_require__.e` 被调用
2. 在内部创建一个Promise, 将这个 `Promise` 的 `resolve ` 和 `reject` 包括这个 `Promise` 返回的对象都挂载到内部的 chunk缓存上.
3. 根据提供的名称, 以及 `publicPath` runtime在内部拼接出 url 到 script 标签上, 向服务器发起脚本请求.
4. 监听 script 标签的完成和失败事件, 做善后处理
5. chunk 会进行下载->执行, 调用 `window['webpackJsonp'].push` 方法, push方法内部会读取 chunk 缓存, 遇到 Promise 会执行它的 `resolve `.
6. resolve后 `then` 被调用, 调用前我们的 chunk 就已经解析完毕, 此时可以使用 `__webpack_require__` 来获取到模块, 然后在第二个 `then` 中读取模块提供的内容.

**图片**:执行流程分析

![import-mulit-file](F:\library\article\assets\import-mulit-file.jpg)