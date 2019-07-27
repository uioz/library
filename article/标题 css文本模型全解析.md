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

那么多出去的高度是从哪里来的, 实际上字体作者可以指定相当多的参数, 其中就包括了字体的上下的空白空间.

具体可以参考[这篇文章](https://juejin.im/post/59c9bc196fb9a00a402e0166), 从制作字体的角度解释了字体的渲染表现.

## line-height 的默认值

> 取决于用户端。桌面浏览器（包括Firefox）使用默认值，约为`1.2`，这取决于元素的 `font-family`。

不过根据我的测试, 字体设计上提供的超出 `em框` 的留白是会被 `line-height` 覆盖的.

无 `line-height` 情况下 "微软雅黑" 字体行框高度为 26.4px, 一旦手动添加 `line-height:1.2` 后行框高度变为 24px.

例子地址:

> https://jsbin.com/dicaqocize/1/edit?html,css,js,output

// TODO 研究有关 匿名盒子的内容

> https://maxdesign.com.au/articles/anonymous-boxes/
>
> https://stackoverflow.com/questions/16823693/inline-anonymous-boxes

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

可以看到不同类型的字体实际上是按照了他们的基线进行对齐的, 由于这几种字体的基线都是 "小写英文字母的底边缘", 即使他们的默认行高不同但是通过基线对齐后它们在**视觉上形成了统一**.

但是一旦选中这些文字我们便察觉到了他们之间的差异:

![1564127184124](C:\Users\zhao\Documents\library\article\assets\1564127184124.png)

# 行布局

🔥🔥🔥💥💥大量术语警告.

行布局实际上要比常提到的块布局复杂的多. 所以在了解行布局前我们需要了解几吨相关的术语, 当然如果你已经了解过了可以直接跳过.

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
  多个连起来的`em框` 组成内容区.
  对于 "替换元素" 来说, 元素的固有高度在加上可能存在的外边距边框或者内边距, 例如: `<img>` 元素是 `display:inline-block` 的, 是可以设置上述那些属性的, 当鼠标拖动过 `<img>` 元素的时候如果 `<img>` 设置了 `margin` 那么 `margin` 也会属于拖动亮起的一部分, 也是 "内容区" 的一部分.

- 行间距

  `line-height` - `font-size` 的差就是行间距(参考 `em框` 小节).
  一般来说行间距可以被平均分为两部分, 一部分在行前一部分在行后.
  可以理解为 word 中的 "段前" "段后"的概念, 不过在 `css` 中只有 "行间距" 无法单独设置 "行前" 和 "行后".

- 行内框
  含有行间距的外边缘就是行内框的大小. 对于 "非替换元素" 来说 `line-height` 大小就是 "行内框" 的高度.
  不过并非只有 "非替换元素" 可以拥有行内框, 当元素是替换元素的时候, 行内框的高度等同于 "替换元素" 内容区高度, 因为**行间距不会应用到替换元素上**.

- 行框

  行框是由一行中最高的行内框和最低的行内框所撑开的.

相当不好理解对吧, 那么为什么有这么多概念呢? 考虑下方的图片:

![1564117066291](C:\Users\zhao\Documents\library\article\assets\1564117066291.png)

在一行文本中可能会存在各种因素影响着文本的显示, 例如上图中存在着多种字体的情况. 而且这张图片中还没有考虑同一行存在 `<img>` 这种替换元素的情况.

不过复杂的设计实际上是为了满足复杂的需要.

## 建立框

为了方便考虑, 我们现在只讨论一行或者多行中只存在 "非替换元素" 和 "匿名文本" 的情况.

我们知道 "`font-size` 确定了内容区域的高度(因为 `font-size` 确定了 `em框` 而多个 `em框` 构成内容区域).

如果一个行内元素的 `font-size` 是 15px, 那么内容区域的高度就是 15px, 且 `em框` 的高度也是 15px:

```html
<span style="font-size:15px">hello world</span>
```

假设我们设置 `line-height` 为 21px, 那么用户代理(浏览器)会通过 `line-height` 减去 `font-size` 然后在除以2, 将这两份高度补到到 `em框` 的上面以及下面此时构成了 "行内框":

![1564139218873](C:\Users\zhao\Documents\library\article\assets\1564139218873.png)

此时我们在段落中填入一个 "异类":

```html
  <p style="font-size:25px;line-height:25px">
    hello world<strong style="font-size:80px">Emmmm.</strong>
  </p>
```

这个例子中 "hello world" 这个匿名文本的 `line-height` 是 25px(line-height有继承特性), 同样的 `<strong>` 标签中的内容也继承了 `line-height` 也就是 25px.

根据 "行内框" 计算规则 `line-height` 减去 `font-size` 除以 2 --> `(25-80)/2 = -27.5` 这些负值被添加到原有的 `em框` 上构成 "行内框".

所以就造成了一种 "行内框" 比 "内容区域" 小的情况, 而且更加有趣的是 **"行内框" 的下边缘实际上会向上提升**, 因为大部分的文字都靠近 `em框` 的中下部分:

![1564142120696](C:\Users\zhao\Documents\library\article\assets\1564142120696.jpg)

在默认情况下文本是按照 "基线对齐" 的, 所以会造成下面的结果, 行框高度超出了 `line-height` 的高度:

![1564140810221](C:\Users\zhao\Documents\library\article\assets\1564140810221.png)

 计算完成 "行内框" 后的 `<strong>` 标签的基线掉出了 "行内框", 由于文本需要按照 "基线对齐" 所以和 "hello world" 文本进行基线对齐, 此时 `<strong>` 的 "行内框" 被整体上抬, 而 "行框" 是根据整行文本中 "行内框" 的最高值和最低值决定的, "hello world" 文本占据了最低值, 而最高值由 `<strong>` 标签占据, 所以 "行框" 的高度比 `line-height` 还要高.

![1564143040611](C:\Users\zhao\Documents\library\article\assets\1564143040611.jpg)

## vertical-height

> [CSS](https://developer.mozilla.org/en-US/docs/CSS) 的属性 **vertical-align** 用来指定行内元素（inline）或表格单元格（table-cell）元素的垂直对齐方式。

`vertical-height` 是一个十分复杂的属性, 其中有几个属性的作用方式是和之前讨论的 "行框模型" 的关系是十分密切的, 包括以下几个属性:

- top

  行内框文本顶端与行框的顶端进行对齐

- bottom

  行内框的底部与行框的底部进行对齐

- text-top

  将元素行内框的顶端与父元素的内容区的顶端对齐

- text-bottom

  将元素行内框的底端与父元素的内容区的底端对齐

- middle

  将元素行内框的垂直中点与父元素基线上的 0.5ex 处对齐

- super

  将元素的内容区和行内框上移. 上移动的距离由用户代理实现.

- sub

  将元素的内容区和行内框下移. 下移动的距离由用户代理实现.

接下来我们来使用 "top" 这个属性来研究一番, 我们在之前的例子中新增一些内容:

```html
  <p style="font-size:25px;line-height:25px">
    hello world
    <strong style="font-size:80px">Emmmm.</strong>
    <span style="vertical-align:top">foobar</span>
  </p>
```

你是否可以解释下方的渲染结果呢:

![1564194377700](C:\Users\zhao\Documents\library\article\assets\1564194377700.png)

我们新添加的 "内联元素" `<span>` 继承了父元素的 `font-size:25px;line-height:25px`, 由于 `<span>` 标签设置了 `vertical-align:top` 所以他的 "行内框" 顶部和我们的 "行框"(蓝色区域) 顶部进行了对齐.

解析图:

![1564195436029](C:\Users\zhao\Documents\library\article\assets\1564195436029.jpg)

- 亮绿色 - 行框
- 红色透明区域 - 行内框区域
- 青色 - 基线
- 蓝色 - 行内框与行框对齐高亮

我们再在原有的基础上加点料:

```html
  <p style="font-size:25px;line-height:25px">
    hello world
    <strong style="font-size:80px">Emmmm.</strong>
    <span style="font-size:40px;vertical-align:top;line-height:10px">foobar</span>
  </p>
```

相信你可以解释下方的结果了吧:

![1564195830449](C:\Users\zhao\Documents\library\article\assets\1564195830449.png)

因为装载 "foobar" 文本的 `<span>` 标签, 被设置了一个较大的字体和一个极小的行高, 文本往往在 `em框` 靠下的位置而经过 "行内框" 生成后, 上下各自减去同样大小的数值, 导致计算完成的 "行内框" 的底部提升, 甚至超过了文字的基线.

使用 `vertical-align:top` 让 `<span>` 的行内框的顶部与行框进行对齐, 导致部分超出 `<span>` 元素的 "行内框" 的文本超出 "行框".



# line-height



# 引用&参考

> https://www.cnblogs.com/youxin/p/3336854.html
>
> https://juejin.im/post/59f5a49951882561a209bd37
>
> https://www.cnblogs.com/shiyou00/p/10690549.html