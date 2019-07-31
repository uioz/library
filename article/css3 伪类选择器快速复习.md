如果说 css 作为前端开发的基本功, 那么 "选择器" 就是基础中的基础. 如果你在复写或者学习这些容易令人混淆的选择器, 那么你就来对地方了, 我的老伙计.

本篇文章会直接了当的比较它们的特性, 帮助你快速的掌握它们:

- first-child
- last-child
- first-of-type
- last-of-type
- only-child
- only-of-type
- nth-child
- nth-last-child
- nth-of-type
- nth-last-of-type

## first-child & last-child

这两个选择器会**匹配一组兄弟元素**中的第一个:

![1564567063286](C:\Users\zhao\Documents\library\article\assets\1564567063286.jpg)

**注意**: 要想使得该选择器起作用实际上需要满足三个条件:

- 被前面的选择器匹配 此例中是 `p`
- 是一组兄弟元素
- 是第一个(或者最后一个)元素

![1564567333831](C:\Users\zhao\Documents\library\article\assets\1564567333831.jpg)

## first-of-type & last-of-type

这两个选择器会匹配同一组类型中的第一个(最后一个)而不理会该元素的位置是否真的是在该组元素的第一个(最后一个):

![1564568928011](C:\Users\zhao\Documents\library\article\assets\1564568928011.jpg)

**注意**: 要想使得该选择器起作用实际上需要满足两个条件:

- 被前面的选择器匹配 此例中是 `p`
- 是一组兄弟元素

![1564568986091](C:\Users\zhao\Documents\library\article\assets\1564568986091.jpg)

## only-child & only-of-type

`only-child` 匹配那些没有兄弟元素的元素, 换句话说匹配那些 "孤儿" 元素:

![1564569860924](C:\Users\zhao\Documents\library\article\assets\1564569860924.jpg)

上图中被 "孤立" 的元素有第一个 `<p>` 和嵌套的 `<span>` 它们都被选择器匹配到了.

`only-of-type` 匹配一组兄弟元素中类型唯一类型的元素:

![1564570350416](C:\Users\zhao\Documents\library\article\assets\1564570350416.jpg)

因为第一个`<p>` 和第二个 `<p>` 以及最后的 `<span>` 在对应的父元素下类型都是唯一的所以它们会被选择器匹配到.

## nth-child & nth-last-child

这些伪类选择器最有意思的一点就是可以传入一个公式 `an+b`, 根据这个公式来匹配元素. 这个公式有很多玩法, 导致有很多人将这个公式的所有组合以及所匹配的内容背下来.

实际上我们的思考方式被 css 给固化了, 因为这个东西从数学的角度来看非常容易摸清楚规律, 例如有如下的代码:

```html
<style>
  p:nth-child(2n+1){
    color:blue;
  }
</style>
<body>
  <p>第一行</p>
  <p>第二行</p>
  <p>第三行</p>
</body>
```

思考模式:

1. 先收集匹配到的元素, 在这个例子中就是三个 `<p>` 标签
2. 从下标 0 后数到 2 表示 `<p>` 的个数, 依次带入公式求值
3. 将对应下标的元素进行匹配(元素下标从1开始数)

结果:

![1564571465216](C:\Users\zhao\Documents\library\article\assets\1564571465216.png)

| 公式            | 解释                               |
| --------------- | ---------------------------------- |
| 2n              | 所有偶数元素                       |
| 2n+1            | 所有奇数元素                       |
| n & n+1         | 所有元素                           |
| n+2             | 第二个元素后的元素(包括第二个元素) |
| n+3             | 第三个元素后的元素(包括第三个元素) |
| 0n              | 啥都匹配不到                       |
| 3n+4            | 4,7,10,13 ....                     |
| 1               | 只匹配第一个元素                   |
| -n+2            | 只匹配前两个元素                   |
| nth-child(odd)  | 奇数元素                           |
| nth-child(even) | 偶数元素                           |

![1564572249317](C:\Users\zhao\Documents\library\article\assets\1564572249317.jpg)

不过不要忘记了 `nth-child` 匹配的依然是同一组兄弟元素, 不过有趣的是 `nth-child` 会利用选择器进行过滤, 但是应用样式的时候却不把样式应用到匹配的元素上:

![1564573844836](C:\Users\zhao\Documents\library\article\assets\1564573844836.jpg)

上图中 `<div>` 中的两组 `<p>` 元素被视为兄弟元素进行匹配, 但是有趣的是作为第三个 `<p>` 元素 "第三行" 也被匹配到了, 这说明在应用样式会直接应用在一组兄弟元素中而不是被匹配到的 `<p>` 元素, 不过需要注意的是如果图片中的 "第三组" 中的 `<p>` 是 `<div>` 的话类型不同样式是不会被应用的.

`nth-last-child` 就是从后向前的版本, 这里就不在详细举例了:

![1564572963078](C:\Users\zhao\Documents\library\article\assets\1564572963078.jpg)

MDN 上还给出了一个有意思的例子, 可以根据元素的数量来控制元素的样式:

```css
li:nth-last-child(n+3),
li:nth-last-child(n+3) ~ li {
  color: red;
}
```

```html
<h4>A list of four items (styled):</h4>
<ol>
  <li>One</li>
  <li>Two</li>
  <li>Three</li>
  <li>Four</li>
</ol>

<h4>A list of two items (unstyled):</h4>
<ol>
  <li>One</li>
  <li>Two</li>
</ol>
```

## nth-of-type & nth-last-of-type

`nth-of-type` 匹配:

- 同一组中相同类型的兄弟元素
- 匹配对应公式计算值的元素

![1564575196264](C:\Users\zhao\Documents\library\article\assets\1564575196264.jpg)

你注意到了吗 `nth-of-type` 和  `nth-child` 是有些区别的, 计算完成后样式的应用到了被匹配的元素身上, 而不是兄弟元素上.

`nth-last-of-type` 是一个从后向前的版本, 这里不在详细介绍:

![1564575569853](C:\Users\zhao\Documents\library\article\assets\1564575569853.jpg)