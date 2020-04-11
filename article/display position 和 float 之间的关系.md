# display position 和 float 之间的关系

> https://drafts.csswg.org/css2/visuren.html#dis-pos-flo

**注意**: 我把 box 翻译成了 "盒子".

这三个属性会影响盒子的生成(box generation)和布局(layout), 它们的作用方式如下:

1. 如果 `display` 被设置为 `none`, 那么 `position` 和 `float` 将不会起作用. 在这种情况下, 该元素不会生成盒子.
2. 否则, 如果 `position` 的值为 `absolute` 或者 `fixed` , 盒子被绝对定位, 那么 `float` 的计算值是 `none` , `display` 的值被设置为下方表格中的值. 盒子的位置由 `top` `left` `right` `bottom` 这四个值和包含块(containing block)所决定.
3. 否则, 如果 `float` 的值不是 `none`, 这个盒子浮动且 `display` 被设置为下方表格中的值.
4. 否则, 如果元素是根元素, `display` 被设置为下方表格中的值, 除了在 css2 中没有定义的 `list-item` 它的计算值没有被定义到底是 `block` 还是 `list-item`.
5. 否则, “display”属性使用给定的值。

| 具体值                                                       | 计算值 **      |
| ------------------------------------------------------------ | -------------- |
| inline-table                                                 | table          |
| inline, table-row-group, table-column, table-column-group, table-header-group, table-footer-group, table-row, table-cell, table-caption, inline-block | block          |
| 其他值                                                       | 和指定的值一致 |

** 所谓的计算值指的是你可以给 `display` 设置一些奇奇怪怪的属性, 这些属性是错误, 但是依然会正常渲染所以浏览器会忽略错误的属性来尝试渲染它们, 也就会计算出一个属性值出现. 对于定位, 浮动, 以及根元素使用 `inline-table` 或者 `inline` 这些具体值的时候, 会根据表格计算.

# 对比 普通流(normal flow) 浮动流(float flow) 以及绝对定位(absolute positioning)

下方提供了几个基于 HTML 的例子来说明区别:

```html
<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01//EN">
<HTML>
  <HEAD>
    <TITLE>Comparison of positioning schemes</TITLE>
  </HEAD>
  <BODY>
    <P>Beginning of body contents.
      <SPAN id="outer"> Start of outer contents.
      <SPAN id="inner"> Inner contents.</SPAN>
      End of outer contents.</SPAN>
      End of body contents.
    </P>
  </BODY>
</HTML>
```

对于这个文档我们假设它使用了如下的规则:

```css
body { display: block; font-size:12px; line-height: 200%; 
       width: 400px; height: 400px }
p    { display: block }
span { display: inline }
```

在每个示例中，由外部元素和内部元素所生成的盒子的最终位置会有所不同. 接下来请看示例.

## 普通流

考虑下方的对于 `inner` 和 `outer` 的样式声明, 它不会修改盒子(boxes)的普通流:

```css
#outer { color: red }
#inner { color: blue }
```

元素 p 包含了所有的内联内容(inline content) 包括:

- 匿名内联文本
- 两个 span

因此所有的内容会在由 `p` 元素建立的包含块的内部所形成的内联格式化上下文(inline formatting context)中进行排列. 看起来像这样:

![Image illustrating the normal flow of text between parent and sibling boxes.](https://drafts.csswg.org/css2/images/flow-generic.png)

**注意**: 图示左侧的数字代表双倍行高下的普通流的位置.

## 相对定位(Relative positioning)

为了查看相对定位的效果, 我们使用下方样式:

```css
#outer { position: relative; top: -12px; color: red }
#inner { position: relative; top: 12px; color: blue }
```

文本流正常的在 `outer` 元素中开始. 然后 `outer` 的文本在第一个行的末尾流入到它在普通流中的位置和尺寸. 随后, 内联盒子包含的文本(被分布在三行中)向上移动了 `-12px` 个单位.

**注意**: 是先使用正常流然后再偏移.

`inner` 中的内容是 `outer` 的子代存在的, 本应随着文字 "of outer contents" (在1.5行). 但是, `inner` 的内容本身相对于 `outer` 内容向下移动了 `12px`, 回到了本应属于它在第二行的位置.

**注意**: 跟随在 `outer` 元素外部的内容不受相对定位的影响.

![Image illustrating the effects of relative positioning on a box's content.](https://drafts.csswg.org/css2/images/flow-relative.png)

**注意**: 如果 `outer` 偏移 `-24px` 个单位, 那么文字会和 `body` 中的文字(1.5行和1行重叠)重叠.

## 浮动的盒子(Floating a box)

考虑使用下方的规则来查看右浮动对 `inner` 元素文本的影响:

```css
#outer { color: red }
#inner { float: right; width: 130px; color: blue }
```

文本流正常的从 inner 盒子开始, 该盒子被从流中拉出然后浮动到右边缘 (其 "width" 已经明确分配). 浮动的行框左侧被缩短, 文档的剩余文字会流入其中.

![Image illustrating the effects of floating a box.](https://drafts.csswg.org/css2/images/flow-float.png)

为了显示 `clear` 属性的效果, 添加了一个兄弟元素:

```html
<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01//EN">
<HTML>
  <HEAD>
    <TITLE>Comparison of positioning schemes II</TITLE>
  </HEAD>
  <BODY>
    <P>Beginning of body contents.
      <SPAN id=outer> Start of outer contents.
      <SPAN id=inner> Inner contents.</SPAN>
      <SPAN id=sibling> Sibling contents.</SPAN>
      End of outer contents.</SPAN>
      End of body contents.
    </P>
  </BODY>
</HTML>
```

使用下列样式:

```css
#inner { float: right; width: 130px; color: blue }
#sibling { color: red }
```

因为 `inner` 盒子和之前一样右浮动, 所以剩余的文本会流入浮动后剩余的空间.

![Image illustrating the effects of floating a box without setting the clear property to control the flow of text around the box.](https://drafts.csswg.org/css2/images/flow-clear.png)

但是, 如果插入的元素使用了 `clear` 并且指定为 `right` (即: 插入元素所生成的盒子不接受右侧挨着浮动元素), 这个插入元素的内容会在浮动元素下方流动:

```css
#inner { float: right; width: 130px; color: blue }
#sibling { clear: right; color: red }
```

![Image illustrating the effects of floating an element with setting the clear property to control the flow of text around the element.](https://drafts.csswg.org/css2/images/flow-clear2.png)

## 绝对定位(absolute positioning)

最后终于到了绝对定位, 请考虑下方 css 声明, 并且考虑它的效果:

```css
#outer { 
    position: absolute; 
    top: 200px; left: 200px; 
    width: 200px; 
    color: red;
}
#inner { color: blue }
```

这会导致 `outer` 盒子的顶部相对于其包含块定位. 对于已经定位的盒子的包含块是通过其最近使用定位的父级来建立的(否则, 则相对于初始包含块, 就像本例). `outer` 盒子的顶边相对于包含块编译 `200px` 左边相对于包含块偏移 `200px`. `outer` 中的盒子相对于其父级正常流动.

![Image illustrating the effects of absolutely positioning a box.](https://drafts.csswg.org/css2/images/flow-absolute.png)

下面的例子展示了一个相对定位的盒子下的绝对定位的子盒子. 虽然 `outer` 盒子不会真正的偏移, 设置 `position:relative` 还意味着这个盒子可以为定位的后代充当包含块. 由于 `outer` 盒子是内联盒子(inline box)它会分为多行, 第一个内联盒子的顶部和左侧的边缘(再图示中由粗虚线表示)充当 `top` 和 `left` 偏移的锚点.

```css
#outer { 
  position: relative; 
  color: red 
}
#inner { 
  position: absolute; 
  top: 200px; left: -100px; 
  height: 130px; width: 130px; 
  color: blue;
}
```

结果如下:

![Image illustrating the effects of absolutely positioning a box with respect to a containing block.](https://drafts.csswg.org/css2/images/flow-abs-rel.png)

如果没有对 `outer` 盒子使用定位:

```css
#outer { color: red }
#inner {
  position: absolute; 
  top: 200px; left: -100px; 
  height: 130px; width: 130px; 
  color: blue;
}
```

那么包含块则是初始包含块(在本例中). 下方图示中展示了这种情况下 `inner` 盒子的最终位置:

![Image illustrating the effects of absolutely positioning a box with respect to a containing block established by a normally positioned parent.](https://drafts.csswg.org/css2/images/flow-static.png)