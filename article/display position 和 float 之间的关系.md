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

```
#outer { color: red }
#inner { color: blue }
```

元素 p 包含了所有的内联内容(inline content) 包括:

- 匿名内联文本
- 两个 span

因此所有的内容会在由 `p` 元素建立的包含块的内部所形成的内联格式化上下文(inline formatting context)中进行排列. 看起来像这样:

![Image illustrating the normal flow of text between parent and sibling boxes.](https://drafts.csswg.org/css2/images/flow-generic.png)

**注意**: 图示左侧的数字代表双倍行高下的普通流的位置.

## 绝对定位(Relative positioning)

