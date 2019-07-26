# 标题: css文本模型全解析

# 前言

在写作本文章前原本打算只是复习一下 `line-height` 和 `vertical-height` 这两个属性而已, 结果发现掉进了一个大坑网上有很多篇文章看的我云里雾里的, 决定从最基础的地方开始进行探索, 这篇文章便是本次探索的一个记录吧.

# 字体

我们先从字体开始, 因为文本布局的基本单位毫无疑问是由文字组成的:

```html
<div>hello world</div>
```

当讨论的文字的时候, 最常见的有关 css 属性就是 `font-size` 了, 通常认为 `font-size` 指定的值就是文字的大小.

想要证明 `font-size` 不等于 "字体大小" 实际上非常简单, 打开浏览器的开发者工具你就可以证明这点, 例如我们有如下的代码:

```html
<body>
    <div style="font-size:25px">hello world</div>
</body>
```

所以文字的高度就是 `25px` 对不对, 但是这是错误的结论:

![1564111940013](C:\Users\zhao\Documents\library\article\assets\1564111940013.png)

上图中我们实际的字体大小是 "32px".

## em框

> 实际上, font-size 属性与你看到的实际字体大小之间的具体关系由字体的设计者来确定. 这种关系设置为字体本身中的一个 em 方框(有人也称为 em 框).
>
> css权威指南(第三版)

你设置的 `font-size` 实际上控制的是文字的 `em框` 而不是字体的本身的大小, 而 `em框` 与文本之间的间距是由字体的设计者决定的. 类似于田字格的效果.

>https://stackoverflow.com/questions/20845711/what-is-a-fonts-em-box-em-unit-and-where-is-it-defined
>
>https://en.wikipedia.org/wiki/Em_(typography)
>
>http://www.d.umn.edu/~lcarlson/csswork/inline/embox.html

不过按照上面的理解, `font-size` 控制的是 `em框` 的大小, 按照这样的设计字体大小至少不会超过 "25px" 才对.

没错实际上大部的字体都严格遵守了 `em框` 的大小限制, 当我们将 `line-height` 大小设置为 `font-size` 一样的时候:

```html
<body style="line-height:25px">
  <div style="font-size:25px">hello world</div>
</body>
```

我们发现字体渲染大小都是小于或者等于 "25px" 的:

![1564112718078](C:\Users\zhao\Documents\library\article\assets\1564112718078.jpg)

那么多出去的高度是从哪里来的, 当然是字体的作者啦, 实际上字体作者可以指定相当多的参数, 其中就包括了字体的上下的空白空间.

具体可以参考[这篇文章](https://juejin.im/post/59c9bc196fb9a00a402e0166), 从制作字体的角度解释了字体的渲染表现.

**题外话**:默认的 `line-height` 可能是由字体作者决定的, 具体请参考MDN的[这篇文章](https://developer.mozilla.org/zh-CN/docs/Web/CSS/line-height)中的内容:

当 `line-height` 没有指定的时候取值为 `normal`, 作用效果正如下方描写的那样:

> 取决于用户端。桌面浏览器（包括Firefox）使用默认值，约为`1.2`，这取决于元素的 `font-family`。

由于 `line-height` 是有继承性质, 一旦你在 body 上进行设置那么这个属性会存在与所有的元素中. 所以**一定要重置默认的 `line-height`** 以减少不同字体不同行高的影响.

## baseline

"baseline" 被称为 "基线", 也是由字体作者定义的, 不过常见的文字基线是英文字母小写 "x"的底边缘.

```html
<body>
  <div style="font-size:25px">
    <span style="font-family:Ubuntu Mono">xYz你好</span>
    <span style="font-family:微软雅黑">xYz你好</span>
    <span style="font-family:宋体">xYz你好</span>
  </div>
</body>
```

![1564113855163](C:\Users\zhao\Documents\library\article\assets\1564113855163.png)

可以看到不同类型的字体实际上是按照了他们的基线进行对齐的, 由于这几种字体的基线都是 "小写英文字母的底边缘", 即使他们的默认行高不同通过基线对齐后它们在视觉上形成了统一.

# 行布局

🔥🔥🔥💥💥大量术语警告.

行布局实际上要比常提到的块布局复杂的多, 复杂的设计实际上是为了满足复杂的需求.

所以在了解行布局前我们需要了解几吨相关的术语, 当然如果你已经了解过了可以直接跳过.

- 行内元素(inline element)
  一个行内元素只占据它对应标签的边框所包含的空间, 默认情况下, 行内元素不会以新行开始, 而块级元素会新起一行.
  也就是常见的 `<span> <a> <em> ...` 等这些 `display:inline` 的元素.

- 匿名文本
  `<p>hello world</p>` 中的 "hello world" 没有被 "行内元素" 包裹起来, 所以它就是 "匿名文本".
  **注意**:不要忘记任何存在于 HTML 中的换行和空格这些实际上都是 "匿名文本".

- 替换元素(replaced element)
  **内容展示**不受 `css` 控制的元素被称为 "替换元素", 例如 `<img> <video> <audio> <iframe> ...`.
  这些元素都有一个特点, 虽然你可以控制其盒模型属性,例如: `width height...` 但是你无法操控其内部的渲染,例如: 你无法控制 `<img>` 显示图片的具体细节.

- 内容区
  多个连起来的`em框` 组成内容区, 张鑫旭老师认为鼠标拖动时候选中文本的背景区域就是内容区域.
  对于 "替换元素" 来说, 元素的固有高度在加上可能存在的外边距边框或者内边距, 例如: `<img>` 元素是 `display:inline-block` 的, 是可以设置上述那些属性的, 当鼠标拖动过 `<img>` 元素的时候如果 `<img>` 设置了 `margin` 那么 `margin` 也会属于拖动亮起的一部分, 也是 "内容区" 的一部分.

- 行间距

  `line-height` - `font-size` 的差就是行间距(参考 `em框` 小节).
  一般来说行间距可以被平均分为两部分, 一部分在行前一部分在行后.
  可以理解为 word 中的 "段前" "段后"的概念, 不过在 `css` 中只有 "端间距" 无法单独设置.

- 行内框
  含有行间距的外边缘就是行内框的大小. 对于 "非替换元素" 来说 `line-height` 大小就是 "行内框" 的高度.
  不过并非只有 "非替换元素" 还可能存在 "替换元素", 当元素是替换元素的时候, 行内框的高度等同于内容区高度, 因为**行间距不会应用到替换元素上**.

- 行框

相当不好理解对吧, 那么为什么有这么多概念呢? 考虑下方的图片:

![1564117066291](C:\Users\zhao\Documents\library\article\assets\1564117066291.png)

当一行中含有多种字体的时候, 这些概念就浮现了出来, 它们都是有理由存在的.

而且这张图片中还没有考虑同一行存在 `<img>` 这种替换元素的情况.

# 引用&参考

> https://www.cnblogs.com/youxin/p/3336854.html
>
> https://juejin.im/post/59f5a49951882561a209bd37
>
> https://www.cnblogs.com/shiyou00/p/10690549.html