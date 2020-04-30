# Vue patch 工作简述

所谓的 patch(补丁) 实际上是一个复杂的流程, Vue 的响应式系统在检查到数据发生变化后会驱动 render 函数, 而 render 函数会生成 VNode 结构.

也就是说一旦数据发生变化后我们就有了两个 VNode 结构, 即数据更新前的 VNode 和数据更新后的 VNode. 在 Vue 实例的首次建立的时候不存在更早的 VNode 此时会执行挂载而不会执行 diff 流程.

所谓的 Patch 实际上执行了多个逻辑:

1. 执行diff 算法
2. 将新的 VNode 与旧的 VNode 进行合并更新
3. 使用更新完成的 VNode 来更新对应的 DOM

所谓的 diff 就是找出新旧两个 VNode 结构的不同, 比对后的结果挂载到 DOM 上, 而 diff 算法会尽可能的减少对真实 DOM 的操作, 里面包含了一系列复杂的流程和算法.

如果重渲染后的 VNode 和旧的 VNode 的根节点不同, 例如先前是一个 `div` 重新渲染后变为了 `p` , 判断的源码如下:

```javascript
function sameVnode (a, b) {
  return (
    a.key === b.key && (
      (
        a.tag === b.tag &&
        a.isComment === b.isComment &&
        isDef(a.data) === isDef(b.data) &&
        sameInputType(a, b)
      ) || (
        isTrue(a.isAsyncPlaceholder) &&
        a.asyncFactory === b.asyncFactory &&
        isUndef(b.asyncFactory.error)
      )
    )
  )
}
```

那么就没有必要执行 patch 逻辑, 直接利用新的 VNode 挂载到 DOM 中.

patch 的流程现在只需要比较先后两次渲染对应的根节点的子节点就可以了, 在 Vue 源码中这个函数叫 `patchVnode` 具体逻辑如下:

1. 如果重渲染后的子节点是文本类型且和旧节点不同则更新对应 DOM 元素的文本
2. 如果旧的子节点和重渲染后的子节点均存在执行**核心 diff 算法**
3. 如果重渲染后存在子节点但是重渲染前不存在, 将重渲染后的子节点追加到 DOM 中
4. 如果重渲染后的子节点不存在, 那么删除旧的 VNode 对应 DOM 的内容
5. 如果重渲染前是文本节点, 重渲染后子节点消失则清空文本节点中的内容

实际上只有**核心 diff 算法**值得我们注意, 也就是重渲染前后组件根节点的子节点都是多个子节点而且子节点发生了变化. 此时 Vue 源码中处理这种情况的函数是 `updateChildren` .

`updateChildren` 要实现的功能是这样的:

1. 如果子节点使用了 key
   1. 找出相同 key 的新旧节点执行 patch 函数(递归调用)
   2. 复用 key 对应的 DOM 元素
2. 如果子节点没有使用 key
   1. 添加新节点到 DOM 中

而实际判断中存在很多问题:

1. 新的子节点数量和旧子节点不匹配
2. 部分使用了 key 而部分没有使用 key
3. 顺序没有发生变化但是渲染的内容改变

所以 `updateChildren` 一定是可以在复杂情况下高效执行的函数, 而针对此种复杂情况 Vue 祭出了**双端排序**来处理此种问题.

实际问题可以简化为如下问题, 给定三个数组 A 和 B 还有 C :

1. 两者内容是随机的
2. 两者长度是随机的
3. B 中的内容可以和 A 中的重复
4. C 的内容和长度和 A 完全一致
5. 无法直接修改 C 可以通过 A 来修改 C

在不调整 A 和 B 的情况下, 将 B 的内容透过 A 的操作按照 B 的顺序完全映射到 C 中去.

A 就是旧的子节点数组, B 就是新的子节点数组, 而 C 就是 DOM, 旧的子节点引用着对应的 DOM, 想要操作 DOM 就得通过旧的子节点, 最后不要忘记了当存在重复的节点的时候对应的 DOM 还得需要复用.

