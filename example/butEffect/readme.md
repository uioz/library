# 结构分析

使用 `button` 元素作为按钮容器, 在内部添加动画.

通过四个 `div` 和定位来模拟边框效果:
```html
<div class="btn-borders">
  <div class="border-top"></div>
  <div class="border-right"></div>
  <div class="border-bottom"></div>
  <div class="border-left"></div>
</div>
```
通过 `span` 来替代直接在 `button` 内部编写文本, 通过 `span` 包裹后可以被选择器命中来使用动画效果.

# 样式分析

样式是通过 "类名" 来赋予样式的, 这样做的优势是灵活性, 和高度的组合性. 难处是如何高效的利用类名来合理的组织样式, 不会使样式重复率高且类名可以被轻松组合.

作者把类名封为了以下几种:
1. button 的直接样式, 清除默认样式赋予基础外观
2. 主体色样式(btn-primary), 内部定义颜色变量值是 HSL 中的色调值
3. 视觉统一样式(btn-ghost), 决定组件颜色的整体协调性
4. 动画样式(btn-border-stroke), 该类名像关联的所有内容都是与动画相关的

# 动画分析

最关键的一共由两点:
1. transform 的使用
2. transition 的使用

## transform 部分

每一个边框在没有状态时使用的样式为:

top:
```css
top: 0;
width: 100%;
height: 1px;
transform: scaleX(0);
transform-origin: left;
```

right:
```css
right: 0;
width: 1px;
height: 100%;
transform: scaleY(0);
transform-origin: bottom;
```

bottom:
```css
bottom: 0;
width: 100%;
height: 1px;
transform: scaleX(0);
transform-origin: left;
```

left:
```css
left: 0;
width: 1px;
height: 100%;
transform: scaleY(0);
transform-origin: bottom;
```

`transform-origin` 有点复杂这里简单的介绍以下, 该属性指定变形时元素对应的原点. 在 css 中坐标从 (0,0) 开始即左上角, 而 `transform-origin` 的值相对于左上角, 初始值为 `center center center` 或者 `0 0 0`, 即变换的原点在元素的中心.  
这也是为什么旋转在默认情况下是中心旋转的原因.  
而 `transform-origin` 的属性可以写多个, 属性的含义由位置决定, 总的来说属性位置规则如下:
1. 定义横向原点位置
2. 定义纵向原点位置
3. 定义 z 轴原点位置(适用于 3D 变换)

而属性值可以使用 数字, 关键字, 百分比, 和全局值, 在上面的例子中使用的是简写方式, 对应的实际值为:
```css
transform-origin: left center;
transform-origin: center bottom;
```

而四个边的宽度都是 `1px` 且通过定位贴合到了内容区的边缘, 而且每一边框 `div` 在默认情况下都将 `scaleY` 和 `scaleX` 的值设置为 0.

实际上当 `scale` 设置为 0 的时候元素将会消失(因为缩小到看不见)而值为 1 的时候元素显示原本的大小.  

而通过指定 scale 和 transform-origin 我们可以控制动画的指向, 例如顶部的边框通过将 `transform-origin` 设置为 `left` 当鼠标浮动到元素上方的时候使用 `scaleX(1)` 就会让元素从左到右显示.  

因为横向的相对原点是在左侧, 横向缩放的时候就会向右边延展, 如果是 `center` 则会向两侧延展.

同理右侧的边框, 纵向原点为 `bottom` 则使用 `scaleY(1)` 的时候就会向上延展.  

## transition 部分

transition 的变化实际上和边框动画执行的顺序息息相关, 当鼠标悬停的时候动画如下展示:
1. 左边框向上延展下边框向右延展
2. 第一步完成后
3. 上边框向右延展右边框向上延展

当鼠标移出后:
1. 上边框向左延展右边框向下延展
2. 第一步完成后
3. 左边框向下延展下边框向左延展

我们看到无论是鼠标移入或者移除都存在一个先后顺序, 在本例中是通过 `transition` 的 `delay` 来控制的. 就拿左边框和上边框来举例:
- 左边框执行动画需要 0.25 秒
- 上边框延迟动画执行 0.25 秒
这样就可以在左边框执行完成后接着执行上边框的动画.  

但是这里有一个问题就是, 鼠标悬浮执行这个逻辑没有毛病, 但是鼠标移出后如果还执行这个逻辑就会造成, 左边框的动画先于上边框完成, 这不是鼠标移除动画执行的逻辑.  

所以在本例中当 `hover` 后才写入适用于 `hover` 的 `transition` 而默认情况下的 `transition` 则是适用于鼠标移出的动画.  

使用左边框和上边框结合代码实际分析, 当鼠标移入时候, 两者的 `transition` 值为:
- `transition: 0.25 cubic-bezier(0.95, 0.05, 0.795, 0.035)`
- `transition: 0.65 0.25 cubic-bezier(0.19, 1, 0.22, 1)`

当鼠标移出时左边框和上边框的 `transition` 值为:
- `transition: 0.65 0.25 cubic-bezier(0.19, 1, 0.22, 1)`
- `transition: 0.25 cubic-bezier(0.95, 0.05, 0.795, 0.035)`

可以看到鼠标进入时上边框延时执行, 而鼠标移出时候左边框延迟执行.  

# 参考

> https://codepen.io/alphardex/pen/pooYKVa
> https://juejin.im/post/5e070cd9f265da33f8653f00#heading-6