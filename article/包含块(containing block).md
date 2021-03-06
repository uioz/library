# 简介

包含块是 css 中的一个很重要的概念, 简单来说一个元素应该在什么位置展示和定位? 应该在包含块内, 那么如何找到给定元素的包含块, 便是该章节讨论的问题.

另外这篇文章不是源规范的标准翻译, 不太重要的部分我就忽略了, 修改了一些表格之类的东西, 还加入了一些我的思考. 如果追求严谨可以去查看[源规范](https://www.w3.org/TR/CSS22/visuren.html#containing-block).

包含块是视觉格式化模型中的内容, 相关规范并不在 level3 规范中而是在 level2 中, level3 规范并没有对此章节进行修改, 而 level2 最新的版本是 css2.2.

# 包含块(containing block)

> https://drafts.csswg.org/css2/visudet.html#containing-block-details

元素所对应的盒子(element's box) 的位置和大小有时是相对于特定的矩形进行计算的, 这个矩形被称为这个元素的包含块. 包含块的定义如下:

1. 根元素所在的包含块是一个被称为初始包含块的矩形.  对于连续媒体，它具有视口(viewport)的尺寸，并锚定在画布原点；它是分页媒体的页面区域。初始包含块的“direction”属性与根元素的“direction”属性相同。 

2. 对于其他元素，如果元素的 `positiion` 是 `relative` 或 `static` ,由离它最近的块容器(block container)的内容区域(content area)的边缘建立.

3. 如果元素的定位使用 `fixed` 包含块对于连续媒体由视口建立, 对于分页媒体由页面区域建立.

4. 如果元素使用了 `absolute` 定位, 则包含块由最近使用了 `absolute` `fixed` `relative` 的父元素通过下列方式建立
   1. 如果该父元素是内联元素(inline element)，则包含块是围绕该父元素的第一个和最后一个内联盒子的 `padding` 盒子生成的边界盒子。 在CSS 2中，如果内联元素分割成了多行，这种情况下包含块没有被定义。 **
   2. 否则就是该父元素的 `padding` 的边缘
   3. 如果不存在使用定位的父元素那么包含块就是初始包含块

** 对应的情况可能入下方代码所示:

```html
<span style='background:aqua;position:relative;padding:10px'>hello <em style="background:white;position:absolute;top:0;left:0;">world</em>!</span>
```

> In paged media, an absolutely positioned element is positioned relative to its containing block ignoring any page breaks (as if the document were continuous). The element may subsequently be broken over several pages.
>
> For absolutely positioned content that resolves to a position on a page other than the page being laid out (the current page), or resolves to a position on the current page which has already been rendered for printing, printers may place the content
>
> - on another location on the current page,
> - on a subsequent page, or
> - may omit it.



在下面的例子中的包含块没有使用任何定位:

```html
<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01//EN">
<HTML>
   <HEAD>
      <TITLE>Illustration of containing blocks</TITLE>
   </HEAD>
   <BODY id="body">
      <DIV id="div1">
      <P id="p1">This is text in the first paragraph...</P>
      <P id="p2">This is text <EM id="em1"> in the 
      <STRONG id="strong1">second</STRONG> paragraph.</EM></P>
      </DIV>
   </BODY>
</HTML>
```

对于页面中的元素, 指出对应的包含块:

| 元素    | 包含块     | 满足条件 |
| ------- | ---------- | -------- |
| html    | 初始包含块 | 根元素   |
| body    | html       | 2        |
| div1    | body       | 2        |
| p1      | div1       | 2        |
| p2      | div1       | 2        |
| em1     | p2         | 2        |
| strong1 | p2         | 2        |

设置以下 css:

```css
#div1 { position: absolute; left: 50px; top: 50px }
#em1  { position: absolute; left: 100px; top: 100px }
```

对于页面中的元素, 指出对应的包含块:

| 元素       | 包含块     | 满足条件 |
| ---------- | ---------- | -------- |
| html       | 初始包含块 | 根元素   |
| body       | html       | 2        |
| div1       | html       | 4.c      |
| p1         | div1       | 2        |
| p2         | div1       | 2        |
| em1        | div1       | 4.b      |
| strong1 ** | em1        | 2        |

** 这里在规范中很模糊至少我没有找到完整的相关描述. 总的来讲 strong1 的包含块是 em1 的原因是 em1 绝对定位了, 而条件 2 的说法是相对于 block container 的, 但是在规范中, 绝对定位元素仅仅被描述为可以建立块级格式化上下文, 而块级格式化上下文并没有说明内部是否必须都是是块级盒子(block-level box), 块级盒子部分也只说明它可以参与块级格式化上下文的建立, 最关键的是 block container 的定义是仅仅能包含块级盒子, 所以没有关键证据指出 em1 此时是 block container.

所以产生一个问题 "containing box 是否就是 block container" ?

## 问题更新

> https://www.w3.org/TR/CSS22/visuren.html#block-boxes

在 css2.2 规范中指明了 block-level 就是 block container.

所以文字现在成为了:

> 问题在于 containing box 是否就是 block container
>
> 以及 containing box 是否就是 block-level box

另外我查阅了最新的 CSS3 规范草案和定位章节目前没有找到合理的解释.