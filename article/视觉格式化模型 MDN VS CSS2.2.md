# 视觉格式化模型(MDN)

本篇文章的绝大部分内容来自于 MDN:

> https://developer.mozilla.org/en-US/docs/Web/CSS/Visual_formatting_model

对应的中文翻译比较旧了本篇文章翻译了英文文章的部分内容.

视觉格式化模型(Visual Formatting Model) 描述了用户代理如何利用文档树 (document tree) 进行处理然后显示到可视媒体上.

再视觉格式化模型中, 再文档树上的每一个元素都会根据和模型(box model) 去生成一个或者多个盒子. 这些盒子的布局(不是盒子内部的布局而是如何布局盒子)受下面条件约束:

- 盒子的尺寸
- 盒子类型
- 定位方案
  - 普通文档流(normal flow)
  - float
  - absolute
- 文档树中元素之间的关系
- 其他的某些外部因素
  - 视口尺寸与位置
  - 所包含的图片的尺寸
  - 以及其他因素

有关视觉格式化模型的大部分信息再 CSS2 中定义, 但是在很多 level 3 规范中都添加了有关信息. 当你阅读相关规范的时候会找到有关 CSS2 规范的引用. 因此再阅读其他布局规范的时候事先了解这个规范将会很有用.

# 视口(viewport)的规则

在连续媒体中，视口是浏览器窗口的可视区域。当视口大小更改时，用户代理可以更改页面的布局-例如: 更改窗口的大小以及移动设备的屏幕方向.

如果视口小于文档的大小，则用户代理需要提供一种能力去滚动到未被显示的那部分文档. 

我们最经常碰见的滚动是在块状区域(block dimension)内滚动-以水平，从上到下的语言中垂直滚动。但是，你也有可能希望在内联区域(inline dimension )中进行同样的操作.

# 盒子生成(Box generation)

盒子生成是视觉格式化模型的一部分, 它从使用文档树中的元素去创建盒子(boxes). 被创建的盒子的类型有很多种, 不同的类型会影响对应的视觉格式(visual formatting). 盒子生成的类型由 CSS `display` 属性决定.

`display` 的最初定义在 CSS2 中在  [CSS Display Module Level 3](https://www.w3.org/TR/css-display-3/) 中被做了扩展, 部分有关 `display` 的术语得到了更新和明确.

CSS 为了完成从文档转为画布上的图像, 需要创建一个中间结构 "盒子树" (box tree) 这种结构用于表示渲染完成的文档.

盒子树中的每个盒子代表其在画布上的空间 和/或 时间的对应元素（或伪元素），而盒子树中的文本同样表示其对应文本节点的内容。

然后对于每一个元素, CSS会生成该元素的 `display` 属性值指定的零个或多个盒子。

> 盒子通常由它的 display 属性所影响, 例如一个拥有 `display:block` 的元素生成的盒子被称为 "block box"(块盒).

# 主盒

当一个元素生成一个或者多个盒子的时候, 其中存在一个主盒(principal box) 包含其后代盒子以及盒子树中的生成内容, 这个盒子也是定位方案中涉及的盒子.

除了主盒子, 某些元素会生成一些额外的盒子, 例如 `display:list-item` 会生成不止一个盒子(其中包括一个主盒子和一个子标记盒子(child market box). 有些 `display` 值会导致元素或者元素的后代不会生成任何盒子. (`none` 和 `contents` 属性)

# 匿名盒子

一个盒子没有对应的HTML元素 ,将创建匿名盒子. 例如: 你创建一个元素声明为 `flex` 然后让它包含了一段文本它没有经过任何标签包裹. 为了修复盒子树一个匿名盒子会被创建去包裹这个文本. 这个匿名盒子的行为就和 flex item 一样. 但是它不能向常规盒子那样被锁定或者被赋予样式因为你无法获取匿名盒子对应的元素.

```css
.flex {
  display: flex;
}

.flex > * {
  background-color: rebeccapurple;
  color: white;
}
```

```html
<div class="flex">
I am wrapped in an anonymous box 
<p>I am in the paragraph</p>
I am wrapped in an anonymous box.
</div>
```

同样的事情也会发生在文本与块元素相交差的时候:

```css
.example > * {
  background-color: rebeccapurple;
  color: white;
}
```

```html
<div class="example">
I am wrapped in an anonymous box 
<p>I am in the paragraph</p>
I am wrapped in an anonymous box.
</div>
```

这会创建三个盒子, 两个匿名盒子一个普通盒子由 `<p>` 标签生成.

有关匿名盒子的一点就是, 匿名盒子会继承父级样式, 但是你无法通过选中它们后去修改样式. 在上面的例子中只有 `<p>` 包裹的文本被赋予了样式.

内联匿名盒子(Inline anonymous boxes) 会在一段文本被内联元素(Inline element) 所分割的时候产生. 例如一段句子中部分文本被 `<em>` 所包裹. 这会创建三个内联盒子(inline boxes)两个内联匿名盒子和在中间的内联匿名盒子. 和匿名盒子一样它们只能继承样式但是无法独立设置样式.

某些格式上下文(formatting context)也会创建匿名盒子. Grid 布局的创建行为和之前的 flex 布局的类似, 将文本字符串转为对应的 匿名 grid item. 多列布局(Multiple-column) 围绕着列创建匿名盒子. 它们同样也无法设置样式或者被锁定. 表格布局将添加匿名盒子以创建合适的表结构, 例如它会添加一个匿名表格行 - 如果不存在一个 `display:table-row` 的盒子的话.

# 行框(Line boxes)

行框是用于包装每一行文本的框. 你可以通过浮动一个元素来观察 "行框" 与它的 "包含块" (containing block) 之间的区别.

在下面的这个例子中我们左浮动了一个 `<div>` 它跟随了一些文字, 它们的父级拥有背景颜色:

```css
.float {
  float: left;
  width: 200px;
  height: 200px;
  background-color: rebeccapurple;
  margin: 20px;
}

.following {
  background-color: #ccc;
}
```

```html
<div class="float"></div>
      <p class="following">This text is following the float, the line boxes are shortened to make room for the float but the box of the element still takes position in normal flow.</p>
```

![image-20200405205507416](./assets\image-20200405205507416.png)

在本例中所谓的 "包含块" 实际上指的是浮动元素和 `<p>` 的父元素, 而跟随浮动后面的文字的行框沿着浮动元素进行了收缩.

# 定位方案和 in-flow 和 out-of-flow 元素

在 CSS 中盒子的铺设方式是由定位方案(Positioning schemes)来决定的:

- normal flow
- floats
- absolute positioning

## 普通流

在普通流中包含:

- 块盒(block boxes)的块级格式化(block-level-formatting)盒子
- 内联盒(inline boxes)的内联级格式化(inline-level formatting)盒子
- 使用了绝对定位或者粘性定位的 块级/内联级 盒子

## 浮动

在浮动模型中，首先根据普通流布置一个盒子，然后从普通流中取出并定位，通常位于左侧或右侧。内容可能会沿着浮动的一侧流动。

## 绝对定位

在绝对定位模型（也包括固定定位）中, 盒子会从普通流中完全移出, 并相对于包含块（在固定定位的情况下是视口）来指定位置。

如果一个元素符合下列条件被称为脱离文档流(out of flow), 反之称为在文档流中(in flow):

- 浮动
- 绝对定位 (sticky absolute fixed)
- 是根元素

# 格式化上下文与 display 属性

可以将盒子(boxes)描述具有外部显示类型(outer display type) `block` 或者 `inline`. 这种外部显示类型指代盒子与盒子间的行为.

盒子同时还具有内部显示类型(inner display type), 决定着它的子代(children)的行为. 对于 普通块 (normal block) 和内联布局 (inline layout) 或者普通流 (normal flow) 它的显示类型是 `flow` 这意味着它的子元素是 `block`  `inline` 二者之一.

但是内部显示属性也可以是 `grid` 或者 `flex`, 在这种情况下它们的直接后代会当作 grid/flex 项显示(grid item / flex item). 在这种情况下元素被称为创建了 `flex` / `grid` 上下文. 在大多数情况下它们的工作方式类似于块级格式化上下文(block formatting context). 但是, 它们的直接后代的行为是 flex/grid 项和普通流中的项目行为不同.

## 独立格式化上下文

元素要不是参与格式化上下文要不就是建立包含块要不就是建立一个独立格式化上下文. 例如: 一个 Gird 容器为其子代建立了 `Grid Formatting Context`.

独立格式化上下文包含:

- 浮动元素
- margin 不会跨格式化上下文穿透

因此创建一个新的块状格式化上下文可以确保浮动元素在盒子的内部. 开发者经常使用 `overflow` 来清除浮动, 而 `overflow` 就会创建一个新的格式化上下文. 在 `display` 中有一个全新的属性 `flow-root` 会创建一个新的格式化上下文而没有任何因为修改 `overflow` 所带来的副作用.

## 块盒(block boxes)

在规范中，在某些位置将块盒(block boxes)，块级盒(block-level boxes)和块容器(block container)都称为块盒. 这几个术语是有区别的, 而块盒(Block boxes) 应该只在没有歧义的情况下使用.

### 块容器(block containers)

块容器要么仅包含参与内联格式化上下文(inline formatting context)的内联级别(inline level)的盒子，要么仅包含参与块格式化上下文(block formatting context)内的块级(block level)盒子。

因此之前讨论自动创建匿名盒子的行为是用于确保所有项目都可以参与 块/内 联格式化上下文. 当元素只包含块级盒子或者内联级盒子的时候这个元素才是一个块容器.

### 块级盒子与内联级盒子

这些分别是块容器内包含的盒子，它们分别参与内联或块布局。

### 块盒子

块盒子是一个块级盒子也是一个块容器. 一个块级盒子同时不是一个块容器(他可能是 flex 或者 grid 容器).

# 总结

块/内联级盒子参与 块/内联 级格式化上下文的创建, 它们的上下文中只能存在对应的盒子.

所谓的 块/内联 级格式化上下文 实际上和所谓的 IE 布局模型类似, 不是所有的元素都需要完整的位置几何等信息, 页面上的内容并非不同的元素内容都是相对于某个元素定位的. 页面中只需要部分元素拥有格式化上下文既可.

普通布局流分为三种, 它们是属于格式化上下文的. 如何理解这句话, 一个 `<div>` 使用 `overflow` 创建块级格式化上下文对吧, 根据上下文来确定对应的文档流:

- 块级格式化上下文
- 内联格式化上下文
- 相对定位

而文档流规定了盒子如何排放.

对了 "块级格式化上下文" 非常常见, 如何建立一个内联格式化上下文呢? 根据规范 一个非替换元素使用 `display:inline` 生成一个内联盒子(inline box).

重点来了 内联盒子 的规范说明是 内联盒子既是 内联级盒子 又包含其内联格式上下文的内容的盒子. 也就是说 `display:inline` 的内部是 "内联格式化上下文" 而它本身也是内联级盒子, 所以可以参与内联格式化上下文. 这说的通因为:

```html
<span>hello <em> world </em></span>
```

是没有问题的.

Except for table boxes, which are described in a later chapter, and replaced elements, a block-level box is also a block container box. A block container box either contains only block-level boxes or establishes an inline formatting context and thus contains only inline-level boxes. Not all block container boxes are block-level boxes: non-replaced inline blocks and non-replaced table cells are block containers but not block-level boxes. Block-level boxes that are also block containers are called block boxes.

The three terms "block-level box," "block container box," and "block box" are sometimes abbreviated as "block" where unambiguous.

# 视觉格式化模型(CSS2.2)

CSS2.2 是 CSS2 规范中的最新标准, 在 Level3 规范中这部分并没有进行大量修改, 依然沿用 CSS2 的标准.

另外标准中的东西太多了, 这里只挑重要的翻译不重要或者和 MDN 重复的部分这里就不翻译了.

## 简介

这章和下章描述了视觉格式化模型: 用户代理如何处理可视媒体的[文档树](https://www.w3.org/TR/CSS22/conform.html#doctree).

在可视化模型中, 文档树上的每一个元素都会根据盒模型生成一个或者多个盒子. 这些盒子的布局由下列因素控制:

- [盒子尺寸](https://www.w3.org/TR/CSS22/box.html#box-dimensions) 和 [类型](https://www.w3.org/TR/CSS22/visuren.html#box-gen).
- [定位方案](https://www.w3.org/TR/CSS22/visuren.html#positioning-scheme) (普通流, 浮动, 以及决定定位).
- 在 [文档树](https://www.w3.org/TR/CSS22/conform.html#doctree) 上的两个元素之间的关系.
- 外部信息 (例如, 视口大小, 来自图片的[固有](https://www.w3.org/TR/CSS22/conform.html#intrinsic)尺寸等).

本章节和接下来的章节所定义的属性对于 [连续媒体](https://www.w3.org/TR/CSS22/media.html#continuous-media-group) 和 [分页媒体](https://www.w3.org/TR/CSS22/media.html#paged-media-group) 都适用. 但是, [外边距属](https://www.w3.org/TR/CSS22/box.html#margin-properties)性应用到分页媒体的时候含义会发生改变(详情请看 [分页模型](https://www.w3.org/TR/CSS22/page.html#page-margins) 了解更多).

The visual formatting model does not specify all aspects of formatting (e.g., it does not specify a letter-spacing algorithm). [Conforming user agents](https://www.w3.org/TR/CSS22/conform.html#conformance) may behave differently for those formatting issues not covered by this specification.

## 视口

用于[连续媒体](https://www.w3.org/TR/CSS22/media.html#continuous-media-group)的用户代理通常为用户提供一个视口(屏幕上的一个窗口或者一片可视区域) 用户可以通过这种方式来浏览文档. 用户代理会在视口缩放后去调整文档的布局(可以参考 [初始包含块](https://www.w3.org/TR/CSS22/visudet.html#containing-block-details)).

当视口小于文档所在绘制的画布大小的时候, 用户代理应该提供一种滚动机制. 每一个画布只能提供一个视口, 但是用户代理可以渲染多个[画布](https://www.w3.org/TR/CSS22/intro.html#canvas)(例如, 一个文档多个视图).

## 包含块

在 css2.2 中, 许多盒子的位置和大小的计算都是与被称为 "包含块" 的矩形盒子息息相关. 通常，生成的盒子充当子盒子的包含块; 我们称为一个盒子为它的后代 "建立" 了包含块. 短语 "盒子的包含块" 指的是 "盒子所在的包含块" 不是它创建的那个.

每一个盒子的位置都是与它的包含块有关, 但是不会被其限制; 它可以溢出到包含块之外.

有关包含块的尺寸如何计算的[细节](https://www.w3.org/TR/CSS22/visudet.html#containing-block-details)请移步[下一章](https://www.w3.org/TR/CSS22/visudet.html).

**译者注**: 包含块可以看我的[另外一篇](https://github.com/uioz/library/blob/master/article/%E5%8C%85%E5%90%AB%E5%9D%97(containing%20block).md)翻译.

## 控制盒子的生成

下面的几节中描述了在 css2.2 中可能生成的几种不同类型的盒子.  盒子的类型在某种程度上影响其在可视化格式模型中的行为. 下面的 `display` 属性指定了盒子的类型.

"display" 属性的某些值可以让源文档的元素生成 "主盒子" 它包含子盒子以及生成的内容, 并且也可以是任意定位方案中所涉及到的盒子. 除了主盒子: "list item"元素外, 某些元素还可能生成其他额外的盒子. 这些额外盒子相对于其主盒子放置.

### 块级元素和块盒子

| 术语                      | 对照翻译         |
| ------------------------- | ---------------- |
| Block-level element       | 块级元素         |
| block boxes               | 块盒子           |
| block-level principal box | 块级主盒子       |
| block formatting context  | 块级格式化上下文 |
| block container box       | 块容器盒子       |
| inline formatting context | 行内格式化上下文 |
| inline-level boxes        | 行内级盒子       |
| block container element   | 块容器元素       |
| non-replaced inline block | 非替换行内块     |
| non-replaced table cell   | 非替换单元格     |
| block boxes               | 块盒子           |

**注意**: 有时候为了简单 "盒子" 会被简写成 "盒".

源文档中的块级元素会按照 "块" 进行视觉格式化(例如 段落). 同时它也是一种生成块级主盒子的元素. `display` 的某些属性值可以让一个元素变成 "块级" 例如 `block` `list-item` 和 `table` . 块级盒子是一种参与块级[格式化上下文](https://www.w3.org/TR/CSS22/visuren.html#block-formatting)的盒子.

在 css2.2 中, 一个块级盒子同样也是块容器盒子除非他是一个表格盒或者是可替换元素的主盒. 块容器盒要么只包含块级盒子要么建立一个行内格式化上下文它只能包含行内级盒子. 一个主盒是一个块容器盒子的元素是一个块容器元素. `display` 属性的某些值可以让非替换元素生成块容器, 例如 `block` `list-item` 和 `inline-block`. 并非所有的块容器盒子都是块级盒子: 非替换行内块和非替换单元格同样是块容器但不属于 "块级". 既是块级盒子又是块容器的情况被叫做 "块盒子".

在没有歧义的地方 "块级盒子" "块容器盒子" 和 "块盒子" 这三个术语被简称为 "块".

#### 匿名块盒子

| 术语                | 对照翻译   |
| ------------------- | ---------- |
| Anonymous block box | 匿名块盒子 |
| in-flow             | 在文档流   |
| inline box          | 行内盒子   |



在文档中像这样的:

```
<DIV>
  Some text
  <P>More text
</DIV>
```

(假设: DIV 和 P 为 `display:block`), DIV 看起来像同时拥有行内内容和块级内容. 为了容易在格式化中定义, 我们假设这里存在了一个围绕着文本的匿名块盒子.

![diagram showing the three boxes for the example above](https://www.w3.org/TR/CSS22/images/anon-block.png)

换句话说: 如果一个块容器(就上上方 DIV 所生成的一样)中包含了一个块级盒子(就像上方的 P 一样), 那么就会强制改块容器只包含块级盒子.

当行内盒包含一个在文档流中的块级盒子的时候, 行内盒子(以及同一个行框的祖先)会围绕着块级盒子(和任何块级连着的或者因空白而折叠导致脱离文档流的以及脱离文档流的兄弟元素)断开, 行内盒子的拆分会形成两个盒子(即使某一边完全是空的), 块级盒子的每一侧都会有一个. 中断前和中断后的行框被包裹在匿名块盒中，块级框盒子成为这些匿名盒子的兄弟盒子. 当行盒收到相对定位的影响的时候, 同样会影响着行盒内部的块级盒子.

**译者注**: 这里 "同一行框的祖先" 可能指的是一个宽度不足一行且被包裹在行框中的行内盒子中存在块级盒子的情况.

如果遵循上述规则的话上述模型适用于下面的例子:

```
p    { display: inline }
span { display: block }
```

与这个 HTML 文档搭配使用:

```
<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01//EN">
<HEAD>
<TITLE>Anonymous text interrupted by a block</TITLE>
</HEAD>
<BODY>
<P>
This is anonymous text before the SPAN.
<SPAN>This is the content of SPAN.</SPAN>
This is anonymous text after the SPAN.
</P>
</BODY>
```

**译者注**: [在线例子](https://jsbin.com/fuyoqalequ/edit?output)

P 元素内部包含匿名文本块 (C1) 和随后的块级元素以及随后的另外的匿名文本块 (C2). 最终的结果盒子相当于 BODY, 包含一个围绕着 C1 的匿名块盒子, SPAN 块盒子, 和另外一个围绕着 C2 的匿名块盒子.

匿名盒子的属性继承自包裹着它的非匿名盒子(例如, 上面例子中的 DIV). 不能继承的属性则使用初始值. 例如, 匿名盒子的字体继承自 DIV 但外边距使用默认值 0.

匿名块盒子生成盒子的元素上设置的属性依然会被应用到改盒子和该元素的内容上. 例如, 上例中的 P 元素如果被设置了边框, 边框会围绕着 C1(开口向行尾) 和 C2 (开口朝行头) 绘制.

**译者注**: 为了更好的理解, 这里附一张图:
![image-20200528160625155](./assets\image-20200528160625155.png)

Some user agents have implemented borders on inlines containing  blocks in other ways, e.g., by wrapping such nested blocks inside  "anonymous line boxes" and thus drawing inline borders around such  boxes. As CSS1 and CSS2 did not define this behavior, CSS1-only and  CSS2-only user agents may implement this alternative model and still  claim conformance to this part of CSS 2.2. This does not apply to UAs  developed after this specification was released.

当解析百分比值的时候, 匿名块盒子会被忽略: 改为使用最近的非匿名祖先盒子. 例如, 如果上方含有匿名块盒子的 DIV 需要使用它的包含块的高度去计算百分比高度时, 它会使用由 DIV 形成的包含块的高度, 不包含匿名块盒子的高度.

### 行内级元素和行内盒子

| 术语                      | 对照翻译         |
| ------------------------- | ---------------- |
| Inline-level element      | 行内级元素       |
| inline box                | 行内盒子         |
| inline-level box          | 行内级盒子       |
| inline formatting context | 行内格式化上下文 |
| atomic inline-level box   | 原子行内级盒子   |

行内级元素是源文档中不构成新内容块的元素; 内容在行中分布(例如, 段落中的强调文本, 行内的图片之类的). 下面的 `display` 属性可以让元素成为行内级 `inline` `inline-table` `inline-block`. 行内级元素生成行内级盒子它可以参与行内格式化上下文.

行内盒子时不仅是行内级盒子而且它的内容参与其自身包含的行内格式化上下文.  一个使用了 `display:inline` 的非替换元素会生成行内盒子. 不是行内盒子的行内级盒子(例如, 行内级替换元素, inline-block 元素, inline-table 元素)被称为原子行内级盒子, 因为它们使用一个不透明框去参与它的行内格式化上下文.

#### 匿名行内盒子

任何直接包含在块容器元素(不是行内元素)必须视为匿名行内元素.

在一个 HTML 标记的文档中就像这样:

```
<p>Some <em>emphasized</em> text</p>
```

`<p>` 生成了一个块盒子, 里面由几个行内盒子. 用于表示强调的盒子是行内盒子通过行内元素 (<em>) 生成. 但是另外的盒子 ("some" 和 "text") 是由块级元素 (`<p>`) 生成的行内盒子. 后者被称为匿名行内盒子, 因为它没有与之相关联的行内级元素.

这种匿名行内盒子从其父块盒子继承可继承属性. 非继承属性使用初始值. 在这个例子中, 颜色会继承自 P, 但是背景颜色使用的是透明值.

**译者注**: 背景颜色无法继承.

随后根据'white-space'属性折叠起来的空白内容不会生成任何匿名行内盒子.

如果上下文很明确, 在本规范中匿名块盒子和匿名行内盒子都会被称为匿名盒子.

当格式化[表格](https://www.w3.org/TR/CSS22/tables.html#anonymous-boxes)的时候会生成更多类型的匿名盒子.

### 总结

元素可以生成盒子, 盒子不是元素.

块容器盒子用于容纳 块级盒子 或者建立 行内格式化上下文, 因此块容器中一旦出现 行内盒子(包括匿名文本) 就会构建行内格式化上下文,  但其中存在的块级盒子会打破按照行排列的规则.

容器约束内部的内容, 块级盒子和行内级盒子约束外部的表现.

块容器盒子有点类似行内盒子, 都需要满足格各自的两种条件. 都像是容器, 其内部有独特的规则.

## 普通流

### 块级格式化上下文

浮动, 绝对定位元素, 不是块盒子的块容器(inline-blocks, table-cells, and table-captions), 和 `overflow` 不是 `visible` 属性的情况, 会为他们的内容建立一个新的块级格式化上下文.

在块级格式化上下文中, 盒子被一个接着一个的垂直摆放, 从包含块的顶部开始. 两个挨着的盒子之间的距离由 `margin` 属性控制. 两个相邻的块级盒子之间的垂直 `margin` 会在格式化上下文中塌陷.

在块级格式化上下文中, 每个盒子的左外边缘与包含块的左边缘相接触. 即使盒子中存在浮动的情况下也依然成立(虽然盒子的行盒可能会因为浮动而收缩), 除非这个盒子也建立一个新的块级格式化上下文(在这种情况下这个盒子会变窄因为里面的浮动元素).

For information about page breaks in paged media, please consult the section on [allowed page breaks](https://www.w3.org/TR/CSS22/page.html#allowed-page-breaks).

> 为什么 BFC 可以阻止 margin 塌陷, 文档已经说明了这一点
>
> 为什么 BFC 可以阻止 float 的溢出, A float is a box 所以 BFC 必须包住它.  must not overlap the margin box of any floats in the same block formatting context as the element itself 规定了浮动元素不可以和 margin 重叠.

// TODO: 继续等等吧, 东西太多了

// TODO: 行盒的规范 https://www.w3.org/TR/css-text-3/#bidi-linebox