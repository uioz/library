# Vue 组件与VNode的初始化

组件 B 依赖组件 A .

在组件 A 初始化的时候实际上组件 B 并不会进行任何处理, 当组件 A 开始渲染的时候 render 函数被调用.

- render 函数中通过 createElement 可以创建组件

此时调用  createComponent 开始创建子组件对应的实例.

createComponent 完成如下的几个步骤:

1. 调用 Vue.extend 通过组件选项和全局选项来创建一个组件构造器
2. 处理父子数据通信 事件监听 props v-model 等
3. 抽象组件和异步组件以及函数组件的特殊处理
4. 利用上述信息新建一个 VNode 实例且返回这个实例

VNode 建立后, Vue 会遍历 VNode 建立对应的 DOM 如果某个节点是组件, 则会实例化 VNode 对应的组件.

组件实例化的过程和 `new Vue()` 的过程一样, 一旦实例化完成后实例会挂载到 `vnode.componentInstance` 上. 然后实例化完成的组件会执行 `$mount` 操作.

后面组件的所有操作都和 `vnode` 无关了, 子组件组件挂载的过程就和上面的步骤一致了.

最后组件初始化完成 父组件的 `vnode` 继续遍历执行上述操作.