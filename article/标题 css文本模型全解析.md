# 标题: css文本模型全解析

# 前言

在写作本文章前原本打算只是复习一下 `line-height` 和 `vertical-height` 这两个属性而已, 结果发现掉进了一个大坑网上有很多篇文章看的我云里雾里的, 决定从最基础的地方开始进行探索, 这篇文章便是本次探索的一个记录吧.

# 内联元素(行内元素)

你也可以直接跳过这章, 去阅读后面精彩的部分, 如果遇到了概念问题可以在回来进行查阅.

我们这里不会仔细的讨论 "内联元素", 只是为了后面的内容做铺垫, 这些内容可能需要你提前预习.

另外 "内联元素" 和 "行内元素" 实际上是同一个东西只是叫法不同, 但是本篇文章中没有统一叫法.

## 表现

一个内联元素只占据它对应标签的边框所包含的空间。

默认情况下，行内元素不会以新行开始，而块级元素会新起一行。

以下的这些元素都是行内元素:

- [b](https://developer.mozilla.org/zh-CN/docs/Web/HTML/Element/b), [big](https://developer.mozilla.org/zh-CN/docs/Web/HTML/Element/big), [i](https://developer.mozilla.org/zh-CN/docs/Web/HTML/Element/i), [small](https://developer.mozilla.org/zh-CN/docs/Web/HTML/Element/small), [tt](https://developer.mozilla.org/zh-CN/docs/Web/HTML/Element/tt)
- [abbr](https://developer.mozilla.org/zh-CN/docs/Web/HTML/Element/abbr), [acronym](https://developer.mozilla.org/zh-CN/docs/Web/HTML/Element/acronym), [cite](https://developer.mozilla.org/zh-CN/docs/Web/HTML/Element/cite), [code](https://developer.mozilla.org/zh-CN/HTML/Element/code), [dfn](https://developer.mozilla.org/zh-CN/docs/Web/HTML/Element/dfn), [em](https://developer.mozilla.org/zh-CN/docs/Web/HTML/Element/em), [kbd](https://developer.mozilla.org/zh-CN/docs/Web/HTML/Element/kbd), [strong](https://developer.mozilla.org/zh-CN/docs/Web/HTML/Element/strong), [samp](https://developer.mozilla.org/zh-CN/docs/Web/HTML/Element/samp), [var](https://developer.mozilla.org/zh-CN/docs/Web/HTML/Element/var)
- [a](https://developer.mozilla.org/zh-CN/docs/Web/HTML/Element/a), [bdo](https://developer.mozilla.org/zh-CN/HTML/Element/bdo), [br](https://developer.mozilla.org/zh-CN/HTML/Element/br), [img](https://developer.mozilla.org/zh-CN/HTML/Element/Img), [map](https://developer.mozilla.org/zh-CN/HTML/Element/map), [object](https://developer.mozilla.org/zh-CN/HTML/Element/object), [q](https://developer.mozilla.org/zh-CN/HTML/Element/q), [script](https://developer.mozilla.org/zh-CN/HTML/Element/Script), [span](https://developer.mozilla.org/zh-CN/HTML/Element/span), [sub](https://developer.mozilla.org/zh-CN/HTML/Element/sub), [sup](https://developer.mozilla.org/zh-CN/HTML/Element/sup)
- [button](https://developer.mozilla.org/zh-CN/docs/Web/HTML/Element/button), [input](https://developer.mozilla.org/zh-CN/HTML/Element/Input), [label](https://developer.mozilla.org/zh-CN/HTML/Element/label), [select](https://developer.mozilla.org/zh-CN/HTML/Element/select), [textarea](https://developer.mozilla.org/zh-CN/HTML/Element/textarea)

## 分类

### 替换元素

> 在 [CSS](https://developer.mozilla.org/zh-CN/docs/Web/CSS) 中，**可替换元素**（**replaced element**）的展现效果不是由 CSS 来控制的。这些元素是一种外部对象，它们外观的渲染，是独立于 CSS 的。

典型的替换元素有:

- `<iframe>`
- `<video>`
- `<embed>`
- `<img>`

### 非替换元素

那些没有 "特异功能" 的 "内联元素" 就是 "非替换元素", 典型值如下:

- `<span>`
- `<em>`
- `<strong>`
- `<i>`

## 布局

内联元素除了 "行内元素不会以新行开始" 这种大家都知道的特性外, 还有几点特殊的表现. 例如块级元素上的几个常见属性:

- margin
- padding
- border

这些属性对于内联元素来说不会影响其**垂直高度**, 但是这不意味着这些内容被裁剪掉了而是因为 `margin` 是透明的而 `padding` 通常来说也是透明的.

唯一例外的是 `border` 它可以被看到, 不过依然不会影响垂直高度:

![1564454615785](C:\Users\zhao\Documents\library\article\assets\1564454615785.png)

图片中文字 "Emmm" 被 `<em>` 包裹了起来并且设置了 `margin` 和 `padding` 和 `border` 请仔细观察在 "垂直方向" 实际上高度并没有增加.

**注意**:这种表现只会出现在 "非替换元素" 上, "替换元素" 的表现是不同的后文中会提到.

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

上图中我们看到一个 `font-size:25px` 的元素却占据了 32.8px 的高度的空间.

## em框

> 实际上, font-size 属性与你看到的实际字体大小之间的具体关系由字体的设计者来确定. 这种关系设置为字体本身中的一个 em 方框(有人也称为 em 框).
>
> css权威指南(第三版)

你设置的 `font-size` 实际上控制的是文字的 `em框` 而不是字体的本身的大小, 而 `em框` 与文本之间的间距是由字体的设计者决定的.

`em框` 的概念就类似小学使用的田字格, 所有字体都相对于田字格进行书写.

>https://stackoverflow.com/questions/20845711/what-is-a-fonts-em-box-em-unit-and-where-is-it-defined
>
>https://en.wikipedia.org/wiki/Em_(typography)
>
>http://www.d.umn.edu/~lcarlson/csswork/inline/embox.html

不过按照上面的理解, `font-size` 控制的是 `em框` 的大小, 按照这样的设计字体大小至少不会超过 "25px" 才对.

没错实际上大部的字体都严格遵守了 `em框` 的大小限制, 当我们将 `line-height` 大小设置为 `font-size` 一样的时候:

// TODO 修改例子

```html
<body style="line-height:25px">
  <div style="font-size:25px">hello world</div>
</body>
```

我们发现字体渲染大小都是小于或者等于 "25px" 的:

![1564112718078](C:\Users\zhao\Documents\library\article\assets\1564112718078.jpg)

那么多出去的高度是从哪里来的?

实际上字体作者有相当大的权力来控制字体,不仅可以让字体大小超过 `em框`, 还可以**指定字体的上下留白空间**.

具体可以参考[这篇文章](https://juejin.im/post/59c9bc196fb9a00a402e0166), 从制作字体的角度解释了字体的渲染表现.

### line-height 的默认值

在MDN上对于 `line-height` 有如下大致的描述:

> 当 line-height 使用默认值的时候(这个值是 normal), 这个值约为 1.2(不同浏览器不同), 取决于元素的 `font-family`.

不过根据我的测试, 字体设计上提供的超出 `em框` 的留白是会被 `line-height` 覆盖的.

固定 `font-size:20px` 且无 `line-height` 情况下 "微软雅黑" 字体行框高度为 26.4px, 一旦手动添加 `line-height:1.2` 后行框高度变为 24px.

例子地址:

> https://jsbin.com/dicaqocize/1/edit?html,css,js,output

## baseline

"baseline" 被称为 "基线", 也是由字体作者定义的, 不过常见的文字基线是该字体的**英文字母小写 "x"的底边缘**, 如果说 `em框` 是田字格, 那么 `baseline` 就是英语练习册上的 "四线三格", 指定了文字对齐的位置, 有如下的例子:

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

我们指定了多种不同的字体, 由于这几种字体的基线都是 "**小写英文字母x的底边缘**", 即使他们的默认行高不同但是通过基线对齐后它们在**视觉上形成了统一**.

但是一旦选中这些文字便可以察觉它们之间的差异:

![1564127184124](C:\Users\zhao\Documents\library\article\assets\1564127184124.png)

// TODO 增加一个基线描述图片

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
  对于 "替换元素" 来说, 元素的固有高度在加上可能存在的外边距边框或者内边距.

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

复杂的设计实际上是为了满足复杂的需要.

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

## vertical-height 实战

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

## line-height 实战

// TODO 简单讲述一下 line-height 属性

在之前的章节中我们知道了 `line-height` 的高度实际上就是 "行内框" 的高度. 如果设置了错误的 `line-height` 的高度可能会带来多行叠加的情况.

当一行中含有不同大小字体的时候我们该如何指定一个良好的 `line-height` 的呢, 例如:

```html
  <p >Lorem ipsum dolor sit amet, consectetur adipisicing elit.
    <strong style="font-size:50px;">big Text</strong>
    dolorum inventore tempora, laboriosam esse a veritatis 
    quae nihil mollitia
  </p>
```

渲染的结果:

![1564372422728](C:\Users\zhao\Documents\library\article\assets\1564372422728.png)

问题所在的原因就是 "行内框" 的高度比 "内容区域" 小, 导致多出去的内容区域渲染的内容溢出. 解决的问题很简单**确保行内框要比内容区域大**就可以了.

我们手动的给大号字体指定一个 `line-height` 其值要大于等于 `font-size`:

```html
  <p >Lorem ipsum dolor sit amet, consectetur adipisicing elit.
    <strong style="font-size:50px;line-height:1em">big Text</strong>
    dolorum inventore tempora, laboriosam esse a veritatis 
    quae nihil mollitia
  </p>
```

或者使用 `line-height` 的缩放因子:

```html
  <p style="line-height:1.5">Lorem ipsum dolor sit amet, consectetur adipisicing elit.
    <strong style="font-size:50px">big Text</strong>
    dolorum inventore tempora, laboriosam esse a veritatis 
    quae nihil mollitia
  </p>
```

由于 `line-height` 具有继承的特性所以 `<p>` 下的所有内容都继承了这个值, 而 "缩放因子" 是根据当前元素的 `font-size` 进行计算的, 所以大号字体会有一个线对其字号 1.5 倍大小的 `line-height` 值.

## 留意 border

不过这里依然存在一些问题, 我们知道 "行内元素" 可以设置 `padding` `margin` 和 `border`, 其中 `margin` 和 `padding` 在垂直方向的值是可以设置也存在但是不会影响到垂直方向的表现.

`border` 和 `margin` 还有 `padding` 的表现一致, 但是有一点不同的是 `border` 大部分情况下都是由颜色的. 如果 `border` 在垂直方向的高度大于 "行间距" 的话, 这样又会造成重叠:

![1564373841012](C:\Users\zhao\Documents\library\article\assets\1564373841012.png)

因此在指定 `line-height` 的时候你还需要囊括 `boder ` 的高度这样才能避免重叠的问题:

```html
  <p style="line-height:1.5">Lorem ipsum dolor sit amet, consectetur adipisicing elit.
    <strong style="font-size:50px;border-top:20px solid;line-height:calc(1.5em + 20px)">big Text</strong>
    dolorum inventore tempora, laboriosam esse a veritatis 
    quae nihil mollitia
  </p>
```

使用 `calc` 可以进行自动计算, 当然手动指定一个高度也是可以的.

## 替换元素

在此之前我们只讨论了 "非替换元素" 的表现, "替换元素" 表现有些区别所以这些进行详细介绍.

不过在此之前, 我们再次回顾一下 "替换元素" 有哪些呢, 常见的有:

- `<iframe>`
- `<video>`
- `<embed>`
- `<img>`

以及 "建立框" 中和 "非替换元素" 稍有不同的两个步骤:

- 内容区

  对于 "替换元素" 来说, 元素的固有高度在加上可能存在的外边距边框或者内边距.

- 行内框

  当元素是替换元素的时候, 行内框的高度等同于 "替换元素" 内容区高度, 因为**行间距不会应用到替换元素上**.

现在开始观察一个实际的例子并且尝试解释布局的原因:

```html
<body>
  <p style="font-size:20px;line-height:1">hello <img src="http://temp.im/100x30" alt=""> world</p>
</body>
```

**图片**:渲染结果:

![1564456170490](C:\Users\zhao\Documents\library\article\assets\1564456170490.png)

根据之前的 "行内框" 的描述, 因为我们的 `<img>` 内容的高度是 30px, 这个高度对于 "替换元素" 来说既是 "内容区" 的高度也是 "行内框" 的高度, 所以这两者的高度都是 30px.

默认情况下 `vertical-align` 的对齐方式是 "baseline"(基线) 对齐, 对于 "替换元素" 来说 "基线" 就是**内容区域的底边缘**, 所以 `<img>` 会和左右两侧的文字进行对齐, 因此我们可以看到 `<img>` 元素下方有留出空白, 实际上是因为对齐而被提升了.

正因为 "基线" 对齐导致 `<img>` 的行内框被提升导致 "行框" 变大才从而间接的导致 "行框" 的高度增加变为了 32.4px 而不是图片的高度 30px, 如下图所示:

![1564457747715](C:\Users\zhao\Documents\library\article\assets\1564457747715.jpg)

- 红色区域 - 文字行内框区域
- 蓝色区域 - 图片行内框区域
- 绿色线 - 基线
- 黑色框 - 行框

"替换元素" 影响了行框的高度但是并不是通过修改父元素的 `line-height` 的完成的, 或者修改自身的 `line-height` 来完成的.

`<img>` 这个 "替换元素" 与其他 "非替换元素" 没有什么不同, 它也会继承 `line-height` 也可以设置 `line-height`, 但是 `line-height` 不会影响到它们(svg会受到影响), 例如我们给 `<img>` 设置一个 100px 的 `line-height` 但是这不会影响到他的 "行内框" 的大小:

```html
  <body>
      <p style="font-size:20px;line-height:1">hello <img src="http://temp.im/100x30" style="line-height:100px;"> world</p>
  </body>
```

修改后的高度依然和之前的例子高度一致.

不过 `line-height` 的值可以被 `vertical-align` 借用, 这个属性的百分比单位是基于 `line-height` 进行计算的, 根据计算的结果在当前的基线上进行上下移动:

```css
img{
  line-height:100px;
  vertical-align:50%;
}
```

这会令 `<img>` 元素相对于基线向上移动 50px.

对了, 不要忘记 "替换元素" 的内容区域是包括了 `margin` `padding` 和 `border` 的, 这点和 "非替换元素" 是有区别的:

![1564458928495](C:\Users\zhao\Documents\library\article\assets\1564458928495.png)

看到了吗垂直方向的高度增加了, 仔细观察你可以看到 `margin-top` 和行框之间存在着一小段距离.

## inline-block



# 引用&参考

> https://www.cnblogs.com/youxin/p/3336854.html
>
> https://juejin.im/post/59f5a49951882561a209bd37
>
> https://www.cnblogs.com/shiyou00/p/10690549.html