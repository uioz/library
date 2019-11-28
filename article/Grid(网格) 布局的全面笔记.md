# 前言

在 MDN 上实际上已经[有了一篇](https://developer.mozilla.org/zh-CN/docs/Web/CSS/CSS_Grid_Layout)非常全面的网格布局的指南, 在我学习了一部分后, 发现网格布局可能是迄今为止最复杂的 `CSS` 概念之一, 其涉及的概念和属性量都是非常繁杂的.

虽然网格布局非常nice, 不过兼容性还是不太乐观, 至少在生产环境中还是不太方便使用, 不过最近几年浏览器的更新速度越来越快, 在不久的将来 `Gird` 布局一定可以成为二维网页布局的不二选择.

为了防止自己以后使用 `Grid` 布局时忘记如何使用的尴尬窘境, 这篇文章写给以后懵逼的我. 为此本篇文章将会讨论 `CSS网格布局规范 Level 1` 规范中的绝大部分内容.

# 简介

网格布局是下一代的布局方式, 用于在页面中提供二维布局, 网格布局是一种流式布局和 `flex` 布局类似, 不过 `flex` 布局被设计用于来控制单行布局.

简单理解 `Gird` 布局是一种控制多行流式布局的方案, 在二维布局方面这套布局方案可以让 "栅格布局" 进入历史的垃圾桶了, 不经如此在原来难以直接实现的各种居中, 流式, 百分比布局都可以直接实现, 而且还可以轻松创建出各种复杂的效果, 如果利用缩写属性以上功能仅用一行代码就可以完成.

# 基本概念

首先**网格布局最主要的目的是用于进行二维布局**, 所谓的二维布局也就是由 "行" 和 "列" 构成, 而网格布局的本质就是控制子元素在网格容器中如何进行排布.

例如在下面的布局是使用 `Grid` 布局完成, 请仔细观察留白缝隙于布局的关系, 该布局实际上是一个 3列 3行的布局.

![1572766821645](./assets\1572766821645.png)

借助于开发者工具, 我们可以清楚的看到 "网格" 布局的 "网格" 是如何体现的:

![1572767045201](./assets\1572767045201.png)

如果使用普通的语言描述, 我们可能会说:

- 元素A 占据了一列三行的区域
- 元素B 占据了三列二行的区域
- 元素C 占据了二列一行的区域
- 元素D 占据了二列一行的区域

而 `Gird` 布局真的允许你这样去定义布局, 其中部分样式代码如下:

```css
.A {
   grid-column-start: 1; /* 第一行开始 */
   grid-column-end: 2; /* 第二行结束 */
   grid-row-start: 1; /* 第一列开始 */
   grid-row-end: 4; /* 第四列结束 */
}
.B {
   grid-column-start: 3;
   grid-column-end: 4;
   grid-row-start: 1;
   grid-row-end: 3;
}
.C {
   grid-column-start: 2;
   grid-column-end: 3;
   grid-row-start: 1;
   grid-row-end: 2;
}
.D {
   grid-column-start: 2;
   grid-column-end: 4;
   grid-row-start: 3;
   grid-row-end: 4;
}
```

上面的子元素声明方式是使用了网格布局中的 "基于线的定位", 这种概念十分简单, 在上面 3*3 网格中, 3列也就对应4条纵向的侧线, 3行也就对应4条横向的侧线, 我们通过定义每一个子元素的:

- 行起点所在的线
- 行终点所在的线
- 列起点所在的线
- 列终点所在的线

来控制子元素所占的空间.

## 网格线

网格线是网格系统中的一个基本单位, 被用于网格容器中的子元素定位, 网格线的概念非常简单, 就是每行或者每列的侧边缘线.

在网格布局初始化后会自动的给网格线根据行和列来添加编号:

![grid](./assets\grid.jpg)

因为中英文都是从左至右书写的文字, 如果在从右至左书写的布局中, 列编号的顺序会被颠倒过来.

## 网格轨道

虽然我们直接可以使用 "行" 和 "列" 来描述网格布局中两条平行水平线和两条平行的垂直线之间的空隙区域, 但是在网格布局中我们使用 "轨道" 来描述它们, 这样一来在原有的 "行" 和 "列" 的概念在网格布局中成为了:

- 行轨道
- 列轨道

轨道有很多有意思的设定, 在深入了解前, 对于轨道来说这里有几个关键行为值得注意.

### 轨道的大小

轨道可以指定大小, 可以指定固定单位, 也可以使用特殊的弹性单位 `fr` 来控制轨道的大小.

**提示**: `fr` 单位的概念类似于 `flex` 布局中的缩放因子.

### 创建额外的轨道来包含元素

> 可以使用网格布局定义一个显式网格，但是根据规范它会处理你加在网格外面的内容，当必要时它会自动增加行和列，它会尽可能多的容纳所有的列。

# 网格容器

想要使用网格布局, 必须声明某个元素为布局容器, 这点和 `flex` 有点类似, 例如:

```html
<div class="wrapper">
  <div class="A">A</div>
  <div class="B">B</div>
  <div class="C">C</div>
  <div class="D">D</div>
</div>
```

```css
.wrapper {
 display: grid;
}
```

在上例中通过给父元素使用 `display:grid` 将 `<div class="wrapper">` 将这个父元素设置为了网格布局容器.

通过给容器元素添加声明后我们可以编写接下来的布局了.

# 布局轨道

### 显式轨道

一个被声明为网格的元素, 很难直接用于布局, 在大部分的情况下我们需要为其设置轨道, 用于容纳子元素.

就像我们第一节中展示的 3*3 布局一样, 图片中的网格状的布局不是因为元素的声明而形成的, 而是由父元素通过声明轨道而形成的:

![1572767045201](./assets\1572767045201.png)

我们通过  `grid-template-columns`和 `grid-template-rows` 来为容器声明轨道的行轨道数和列轨道数, 声明一个 3*3 的轨道的代码如下:

```css
grid-template-columns:1fr 1fr 1fr; /* 列轨道数 */
grid-template-rows:200px 200px 200px; /* 行轨道数 */
```

这里展示了 `grid-template-columns`和 `grid-template-rows` 的两个有趣的点:

1. 所见即所得 - 即声明中存在多少个单位即存在多少个轨道
2. 可以使用混合单位 - 在上例中我们使用了新的 `fr` 单位和传统的 `px` 单位

**`fr` 单位:**

这里稍微提及一点新的 `fr` 单位, 在随后的章节中会全面的介绍新的 `fr` 单位.

> 轨道可以使用任何长度单位进行定义。 网格还引入了一个另外的长度单位来帮助我们创建灵活的网格轨道。新的`fr`单位代表网格容器中可用空间的一等份。下一个网格定义将创建三个相等宽度的轨道，这些轨道会随着可用空间增长和收缩。

实际上 `fr`单位的工作机制和 `flex` 布局中的缩放因子是类似的.

在这个例子中容器的列轨道被分为了 4 列, 其中第一列元素占据了 2 个空间, 第二个以及第三个元素各占据 1 个空间的大小:

```css
.wrapper {
  display: grid;
  grid-template-columns: 2fr 1fr 1fr;
}
```

在这个例子中, 展示了混合单位的声明, 第一列被固定为 500px , 第二列轨道占据剩余空间 3 份中的 1 份, 第三列轨道占据 3 份中的 2 份:

```css
.wrapper {
  display: grid;
  grid-template-columns: 500px 1fr 2fr;
}
```

回到之前的例子中, 这两句话:

```css
grid-template-columns:1fr 1fr 1fr; /* 列轨道数 */
grid-template-rows:200px 200px 200px; /* 行轨道数 */
```

声明了一个 3 列自动适应轨道宽度以及 3 行固定高度为 200px 的轨道布局.

### 隐式轨道

除了使用  `grid-template-columns`和 `grid-template-rows`  外我们还可以使用:

- `grid-auto-rows`
- `grid-auto-columns`

来定义隐式轨道, 这里我们使用 `grid-auto-rows` 来举例, 通过下列代码我我们声明了一个 2 * 1 的网格布局:

```css
.wrapper{
  display:grid;
  grid-gap:10px;
  grid-template-columns: 1fr 1fr;
  grid-template-rows:100px;
}
```

但是对应的子元素却有四个:

```html
<div class="wrapper">
  <p>a</p>
  <p>b</p>
  <p>c</p>
  <p>d</p>
</div>
```

那么第二行的 c 和 d 元素所在的轨道是没有指定高度的, 那么网格会创建隐式轨道去存放这些溢出的元素.

![1574819509926](C:\Users\zhao\Documents\library\article\assets\1574819509926.png)

`grid-auto-rows` 的默认值是 `auto` 它控制了隐式轨道行的高度, 现在为其添加一个固定值.

```css
.wrapper{
  display:grid;
  grid-gap:10px;
  grid-template-columns: 1fr 1fr;
  grid-template-rows:100px;
  grid-auto-rows:50px;
}
```

然后在添加几个子元素:

```html
  <div class="wrapper">
    <p>a</p>
    <p>b</p>
    <p>c</p>
    <p>d</p>
    <p>f</p>
    <p>g</p>
  </div>
```

我们可以看到新的隐式轨道的高度都是固定值 50px 了, 当然这两个属性可以设置为其他属性值, 这里就不再展开了.

![1574820737750](C:\Users\zhao\Documents\library\article\assets\1574820737750.png)

### 轨道单位

#### 数量单位 repeat

`repeat` 是用于简化轨道编写的一个属性, 例如我们要定义三个轨道, 利用我们之前的写法, 定义三列轨道代码如下:

```css
grid-template-columns: 1fr 1fr 1fr;
```

使用 `repeat` 可以表示为:

```css
grid-template-columns: repeat(3, 1fr);
```

它还可以用于表示轨道数量的一部分:

```css
grid-template-columns: 20px repeat(3, 1fr) 20px;
/* 等同于 */
grid-template-columns: 20px 1fr 1fr 1fr 20px;
```

在 `repeat` 后续的参数还可以用于重复轨道模式:

```css
grid-template-columns: repeat(2, 1fr 2fr);
/* 等同于 */
grid-template-columns: 1fr 2fr 1fr 2fr;
/* 另外一个例子 */
grid-template-columns: repeat(2, 1fr 2fr 3fr);
/* 等同于 */
grid-template-columns: 1fr 2fr 3fr 1fr 2fr 3fr;
```

#### 大小单位 minmax

`minmax` 单位用于指定轨道的 "高度" 以及 "宽度", 如同它的名字一样, `minmax` 中包含了两个值:

- min 最小值
- max 最大值

就拿最常用的行高来将, `minmax` 单位可以同时指定行轨道的 "最小值" 和 "最大值":

```css
/* 轨道行内的内容高度低于 100px 的时候, 轨道高度为最小值 100px */
/* 轨道行内的内容高度高于 200px 的时候, 轨道高度为最大值 200px */
grid-auto-rows: minmax(100px, 200px);

/* 轨道行内的内容高度低于 100px 的时候, 轨道高度为最小值 100px */
/* 轨道行内的内容高度高于 100px 的时候, 轨道高度会扩展到可以容纳最大元素的高度 */
grid-auto-rows: minmax(100px, auto);
```

# 新的关键字

- flex
- max-content
- min-content
- fit-content

# 网格布局

## 线布局

线布局的概念非常简单, 在第一节中我们已经讨论了过了它, 在这里我们来详细的讨论一下他的其他的工作特性.

一个网格中的区域我们称为 "网格单元", 那么如何确定一个 "网格单元" 在网格中的位置于大小呢?

在线布局中我们通过 "网格单元" 的四条边相对网格来进行定位, 网格布局中提供了我们一组相关的属性:

```css
.A {
   grid-column-start: 1; /* 第一行开始 */
   grid-column-end: 2; /* 第二行结束 */
   grid-row-start: 1; /* 第一列开始 */
   grid-row-end: 4; /* 第四列结束 */
}
```



## 缩写



## 模板布局

## 命名线布局

## 缩写格式



