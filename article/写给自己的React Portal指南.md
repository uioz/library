# 我为什么需要 Portal

一个表单页面, 当用户点击提交表单的时候我们会弹出一个模态框用于提示是否提交.

大多数情况下, 这个模态框弹出的逻辑和模态框 DOM 本身都是在嵌套多层的组件内部编写的, 很有可能就是在提交按钮内部, 此时有可能出现这种情况:

![需要模态框](F:\library\article\assets\需要模态框.jpg)

然后我们在 `li` 中弹出一个全局覆盖的模态框:

![被限制的模态框](F:\library\article\assets\被限制的模态框.jpg)

我们的全局模态框被限制在了 `li` 中, 我们只能使用 `position:fixed` 的方式来让子元素**跳出父元素的控制**.

想要摆脱被禁锢在父元素内部的控制, 我们可以借助于 `Portal` API.

# 基本概念

`Portal` API 可以让 React 组件挂载到任何 DOM 下, 在上面的例子中, 我想把模态框容器挂载到 `body` 元素下, 这样就可以不使用蹩脚的 `position:fixed` 跳出父元素.

而且最有意思的是, 虽然 DOM 结构上 `body->模态框` 和 `div->ul->li->模态框` 两者之间没有任何关系.

但是在 React 中建立的父子关系依然可以保留, 例如:

- 在模态框被点击后事件会冒泡到 React 父组件中

- 父组件依然可以通过 Props 向模态框传递数据

  

# 使用方式

`Portal` API 在 `ReactDOM` 下:

```javascript
import ReactDOM from 'react-dom';
ReactDOM.createPortal();
```



