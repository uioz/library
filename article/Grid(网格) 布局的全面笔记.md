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
   grid-column-start: 1; /* 第一列开始 */
   grid-column-end: 2; /* 第二列结束 */
   grid-row-start: 1; /* 第一行开始 */
   grid-row-end: 4; /* 第四行结束 */
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

"网格单元" 是网格布局中的基本单位一个 3*3 的网格中存在 9 个网格单元, 而多个 "网格单元" 构成 "网格区域", 那么让多个网格单元变为网格区域呢?

在网格布局中我们定义的是轨道, 而轨道两侧的内容就是线, 借用之前的一张描述 3*3 轨道图片:

![grid](./assets\grid.jpg)

我们看到 3行或者3列轨道会存在四条线, 这些线从1开始按行和列编号, 所以 "线" 也被称为 "编号线", 通过网格区域四条边缘线我们就可以进行定位一片区域, 网格布局中提供了我们一组相关的属性:

```css
p:first-child{
   grid-column-start: 1; /* 第一列开始 */
   grid-column-end: 2; /* 第二列结束 */
   grid-row-start: 1; /* 第一行开始 */
   grid-row-end: 4; /* 第四行结束 */
}

p{
  padding:10px;
  background:aqua;
}

.wrapper{
  display:grid;
  grid-gap:5px;
  grid-template-columns:repeat(3,1fr);
  grid-template-rows:repeat(3,1fr);
}

*{
  margin:0;
}
```

```html
  <div class="wrapper">
    <p>1</p>
    <p>2</p>
    <p>3</p>
    <p>4</p>
    <p>5</p>
    <p>6</p>
    <p>7</p>
  </div>
```

结果:

![1575423817939](./assets\1575423817939.png)

### 反方向计数

编号线可以指定负值, 从左向右书写的语言中, 默认的编号线是 1, 而对应的 -1 则是最右侧的编号线,  -2 就是右侧向左数的第二条编号线.

作用在行上的编号线同理, 在一个 2 * 2 的网格布局中, 左上角的网格区域表示如下:

```css
grid-column-start: 1; /* 第一列开始 */
grid-column-end: 2; /* 第二列结束 */
grid-row-start: 1; /* 第一行开始 */
grid-row-end: 2; /* 第四行结束 */
```

如果我们使用负值:

```css
grid-column-start: -1; /* 最右边的线 */
grid-column-end: -2; /* 第二右边的线 */
grid-row-start: -1; /* 最后一个行线 */
grid-row-end: -2; /* 倒数第二行线 */
```

结果整个布局就被反转了过来, 原本的左上的定位变成了右下.

在 2 * 2 的网格布局中, 行和列方向上都有 3 条线, 而 3 和 -1 都代表这最后一条线, 利用这点, 在任意轨道量的布局中, 使用:

```css
grid-column-start: 1; /* 列起始线 */
grid-column-end: -1; /* 列结束线 */
grid-row-start: 1; /* 行起始线 */
grid-row-end: -1; /* 行结束线 */
```

都可以获取行和列的最大跨度.

### 缩写

首先使用编号线定位的时候, 我们指定了线的开始, 可以不指定结束, 这种情况下编号线会自动定位在下一行或者下一列用于结束(下一列的概念取决于书写方向)这种行为被称为 "默认跨度":

```css
/* 只定义开始 */
grid-column-start: 1;
grid-row-start: 1;

/* 等同于 */
grid-column-start: 1;
grid-column-end: 2;
grid-row-start: 1;
grid-row-end: 2;

/* 只定义开始 */
grid-column-start: 3;
grid-row-start: 3;

/* 等同于 */
grid-column-start: 2;
grid-column-end: 3;
grid-row-start: 3;
grid-row-end: 4;
```

#### `grid-column` `grid-row` 缩写属性

`grid-column-start` 和 `grid-column-end` 属性可以合并为 `grid-column`, `grid-row-start` 和 `grid-row-end` 则合并为 `grid-row`.

例如:

```css
/* 完整写法 */
grid-column-start: 1;
grid-column-end: 2;

/* 缩写 */
grid-column: 1 / 2;

/* 完整写法 */
grid-column-start: 2;
grid-column-end: 3;

/* 缩写 */
grid-column: 2 / 3;
```

`grid-row` 同理这里我们就不在展开了.

利用缩写属性的同时我们还可以继续利用默认跨度:

```css
/* 只定义行开始 默认的 grid-row-end 是 2 */
grid-row-start: 1;
/* "缩写属性" + "默认跨度" */
grid-row: 1;

/* 只定义列开始 默认的 grid-column-end 是 3 */
grid-column-start: 2;
/* "缩写属性" + "默认跨度" */
grid-column: 2;
```

#### `grid-area` 属性

`grid-area` 属性更进一步允许将四个属性合并到一个属性中完成, 这四个属性的顺序如下:

- grid-row-start
- grid-column-start
- grid-row-end
- grid-column-end

```css
/* 一气呵成 */
grid-area: 1 / 1 / 4 / 2;

/* 完整写法 */
grid-column-start: 1; /* 第一列开始 */
grid-column-end: 2; /* 第二列结束 */
grid-row-start: 1; /* 第一行开始 */
grid-row-end: 4; /* 第四行结束 */
```

我们使用 `margin` 缩写, 是通过顶部开始然后顺时针定义各个边距, 在 `grid-area` 只要记住他是逆时针就可以了.

### span 关键字

不要忘记了我们定义网格布局的时候指定的是 "轨道" 数量, 而不是编号线的数量.

所以除了使用 "编号线" 也就是轨道的侧边来布局, 还可以通过指定 "轨道" 的数量来进行布局而且这两者还可以进行结合:

```css
grid-column-start: 2; /* 从第二个编号线开始 */
grid-column-end: span 3; /* 横跨 3 列结束, 也就是编号为 5 */
```

```html
  <div class="wrapper">
    <p>1</p>
    <p>2</p>
    <p>3</p>
    <p>4</p>
    <p>5</p>
    <p>6</p>
  </div>
```

```css
.wrapper{
  display: grid;
  grid-template-columns: repeat(9,1fr);
}

p:first-child{
  background:aqua;
  grid-column-start: 2; /* 从第二个编号线开始 */
  grid-column-end: span 3; /* 横跨 3 列轨道结束, 也就是编号为 5 */
}

*{
  margin:0;
}
```

结果:

![1575512352746](./assets\1575512352746.png)

我们将 css 修改一下:

```css
grid-column: span 2 / 5; /* 横跨 2 列轨道开始, 第 5 个编号线结束*/
```

![1575512699461](./assets\1575512699461.png)

但是指定轨道数量, 无法告诉网格区域的 "起始" 和 "结束" 到底在哪里, 所以在混合使用的时候, 网格区域的定位表现是由 "网格线" 来决定的, 修改 css 代码:

```css
grid-column: span 2 / 2;
```

结果:

![1575512897533](./assets\1575512897533.png)

由于网格线的结束线是 2 , 是完全无法容纳 2 个列轨道的, 但是指定了结束编号线, 那么 列轨道 会自动的适应编号线的定位.

## 模板布局

网格模板区域允许你对一片区域进行命名, 而命名后的 "区域" 就被称为模板.

```css
grid-area: template; /* 声明一个 template 模板区域 */ 
grid-area: main; /* 声明一个 main 模板区域 */ 
```

和字面意思一样既然叫做模板, 因该是可以在某个地方使用的:

```css
grid-template-areas: 
      "template template template"
      "main     main     main"; /* 2*3 的布局 */
```

这应该是 css 中你见过的最诡异的语法, 请仔细观察上面的例子, 虽然语法奇特但是却不难理解, 因为 `grid-template-areas` 属性中提供的参数与 `grid-area` 提供的元素命名有着强烈的映射关系.

接下来我们来看一个完整的例子, 我们要建立一个 "页面", 这个页面有三个区域:

- 页头一般叫做 header
- 内容区域一般叫做 main
- 页尾一般叫做 footer

除了内容区域需要有3列外, 其余的均占满一行, 我们先来构建布局:

```html
<body>
  <header>header</header>
    <div class="main1">main</div>
    <div class="main2">main</div>
    <div class="main3">main</div>
  <footer>footer</footer>
</body>
```

其次在编写样式:

```css
header,footer,div{
  background:aqua;
}

body{
  display:grid;
  grid-template-columns: repeat(3, 1fr);
  grid-auto-rows: minmax(100px, auto);
  grid-gap:5px;
  /* 使用网格模板 */ 
  grid-template-areas:
    "header header header"
    "main1 main2 main3"
    "footer footer footer"
}

/* 声明网格模板 */ 
header{
  grid-area:header;
}

.main1{
  grid-area: main1;
}

.main2{
  grid-area:main2;
}

.main3:{
  grid-area:main3;
}

footer{
  grid-area:footer;
}
```

结果:

![1575856756995](./assets\1575856756995.png)

#### 空白网格单元

有的时候我们希望在网格布局中留出空白的单元格, 而不是使用网格区域进行填充, 你需要在 `grid-template-areas` 上使用 `.` 点号即可, 继续上面的例子我们来实现添加一个空行:

```css
  grid-template-areas:
    "header header header"
    "main1 main2 main3"
    ". . ."
    "footer footer footer"
```

效果:

![1575858383656](./assets\1575858383656.png)

#### 跨域多个网格单元

概念和 `excel` 中的合并单元格类似, 在模板布局中实现起来也异常简单, 下面的例子中我简单的重复几次命名使得 `main1` 和 `main2` 构成了一个矩形区域:

```css
  grid-template-areas:
    "header header header"
    "main1  main2  main2"
    "main1  main2  main2"
    "footer footer footer"
```

模板布局就会映射到页面布局:
![1576203343814](./assets\1576203343814.png)

#### `grid-template` 显式网格简写

`grid-template` 它可以允许你在一次编写中指定多个属性包含:

- grid-template-rows
- grid-template-columns
- grid-template-areas

也就是说可以在一个属性中指定轨道高度轨道列数以及模板布局, 他被称为 **显示网格简写** 是因为它无法控制创建隐式轨道的行为.

```css
    grid-template: 
      "header header header header" minmax(100px, auto)
      "main1  main1  main2  main3" minmax(100px, auto)
      "footer footer footer footer" minmax(100px, auto)
             / 1fr 1fr 1fr 1fr;          
```

在每一行的末尾我们指定了该轨道行高, 在整个属性的末尾通过 `/` 进行分割后指定轨道列数以及宽度.

#### `grid` 简写

这个简写比 `grid-template` 还要更进一步, 它还允许你控制隐式轨道, 这个属性在使用的过程中会将 `grid-gap` 设置为 0, 且无法在该简写中指定这个属性.

该简写相当于:

- grid-template-rows
- grid-template-columns
- grid-template-areas
- grid-auto-rows
- grid-auto-columns
- grid-auto-flow

TODO: 继续编写

## 命名线布局

在 "编号线" 一节中我们使用了数字作为编号, 来控制网格区域的大小. 线布局中不仅仅可以使用编号, 还可以为其命名, 然后利用命名来进行布局.

### 模板布局隐式定义的命名线

在我们使用模板布局的时候会自动创建命名线, 现在我们先来观察一下命名线生成的规则, 下方的代码通过模板布局生成了一个 2*2 的布局:

```html
<body>
  <div class="element1">element1</div>
  <div class="element2">element2</div>
  <div class="element3">element3</div>
  <div class="element4">element4</div>
</body>
```

在 css 中我给四个单元格进行命名:

```css
header,footer,div{
  background:aqua;
}

body{
  display:grid;
  grid-template-columns: repeat(2, 1fr);
  grid-auto-rows: minmax(100px, auto);
  grid-gap:5px;
  grid-template-areas:
    "element1 element2"
    "element3 element4"
}
/* 命名在这里 */
.element1{
  grid-area:element1;
}

.element2{
  grid-area:element2;
}

.element3{
  grid-area:element3;
}

.element4{
  grid-area:element4;
}
```

布局结果(含对应的命名线):

![1576304156181](C:\Users\zhao\Documents\library\article\assets\1576304156181.png)

从上图可以看到网格布局给网格区域自动分配了 "命名线", 分配命名线的规则就是获区域的命名, 然后为其添加 `-start` 来表示开始, 添加 `-end` 表示结束.



## 缩写格式



# 其他属性

