# 标题: 重温 vertical-align

# vertical-align 使用前提

> [CSS](https://developer.mozilla.org/en-US/docs/CSS) 的属性 **vertical-align** 用来指定行内元素（inline）或表格单元格（table-cell）元素的垂直对齐方式。
>
> https://developer.mozilla.org/zh-CN/docs/Web/CSS/vertical-align

1. 常见的 inline 元素
   1. inline: `<img>` `<span>` `<strong>` `<em>` 未知元素 ...
   2. inline-block: `<input>`(ie8+) `<button>`(ie8+) ...

2. 常见的 table-cell 元素
   1. 只有 `<td>`.

# vertical-align 支持的单位

1. 线类
   - baseline(默认值)
   - top
   - middle
   - bottom
2. 文本类
   - text-top
   - text-bottom
3. 上标下标类
   - sub
   - super
4. 数值类
   - 20px
   - 2em
   - 20%
   - ...
5. 关键字
   1. inhert
   2. initial
   3. unset

## 数值单位

本节提到的数值实际上就是常说的 `css单位`, 包括几种绝对单位和相对单位.

 数值单位有以下几种特点:

1. 单位中含有数字
2. 数字支持负值

**表现方式**: 在 baseline 对齐的基础上上下偏移对应的数值大小.

**注意**: 百分比值的大小是相对于行高来进行计算的, 在下面这个例子由于我们指定了 `0px` 的行高, 所以 `vertical-align` 不会起任何作用.

```css
div {
    line-height:0px;
    vertical-align:40%;
}
```

# vertical-align 与 line-height 的关系

在页面的任意文本布局中除非手动修改, 文本默认情况下都受到了`vertical-align` 和 `line-height` 的影响.

例如如下代码:

```html
<div style="background:#000">
    <img src="http://temp.im/640x260/fff" alt="">
</div>
```

在图片的下边缘会出现多余的空白:

> https://jsbin.com/gepejosicu/1/edit?html,output

因为 `vertical-align` 默认为基线对齐, 也就是字母 `x` 的下边缘, 我们添加一个 `x` 到图片的后面可以看到图片对齐到哪里去了:

> https://jsbin.com/qexuvunuyi/1/edit?html,output

我们有很多种方式来解决这个 "多余高度" 的问题:

1. 改为 `display:block` 因为 `vertical-align` 不会对 `block` 起作用
2. 修改 `vertical-align`  为 `bottom` 或者 `top` 相对于该行顶部与底部进行对齐
3. 修改 `line-height` 为 0 , 一旦为 0 则原本的 "基线" 就会向上移动
4. 修改 `font-size` 为 0 , 一旦修改为 0 文字相当于不存在, 而基线也就相当于不存在了

# 实战

## 同一行的不定高度文本与图片垂直居中

```html
<div class="test-list">
    <span>请酌情复制为多行文本</span>
    <img src="xxx.jpg">
</div>
```

```css
.test-list > span {
    display:inline-block;
    width:200px;
    vertical-align:middle;
}

.test-list > img {
    vertical-align:middle;
}
```

## 图片与文字垂直居中

