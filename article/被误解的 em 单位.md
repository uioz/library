#  被误解的 em 单位

`em` 单位往往被认为是相对于父元素的  `font-size ` 大小来进行计算的.

例如一个元素的父元素设置了如下的 `font-size`:

```css
body {
    font-size:20px;
}
```

而其中的 `div` 设置了 `font-size` 设置了 `1.5em` :

```css
div {
    font-size:1.5em;
}
```

那么 `div` 元素的 `font-size = 20 * 1.5 = ` **30px**.

![firefox控制台](F:\library\article\assets\1559825791859.png)

# 实际情况

我们都知道 `em` 是相对单位, 但是实际上 `em` 单位相对的是自身的 `font-size` 大小, 我们可以尝试一下:

```css
div {
    font-size:20px;
    margin:2em;
}
```

![firefox控制台](F:\library\article\assets\1559825856377.png)

利用这一点, 我们来打破经典的 **`em` 是相对于父元素大小的观点**:

```css
body {
    font-size:20px;
}
div {
    font-size:20px;
    margin:2em; /* 我可没有相对于 body 的 font-size 来进行计算 */
}
```

![firefox控制台](F:\library\article\assets\1559825939539.png)

如果继承父元素的 `font-size` 的大小, `div` 的 `margin` 计算值是 `60px` 而不是 最后的 `20px` , 显然这个值是根据 `div` 本身的 `font-size` 来进行计算的.

所以在此之前我们所了解的:

> em 单位是相对于父元素的 font-size 来计算的

应该改成:

> em 单位是相对于元素的 font-size 来计算的, 但是 font-size 会继承父元素的大小

而日常开发中 `em` 单位常见于字体单位的而不是其他属性的单位, 所以无法看出 `em` 实际上是相对于元素的 `font-size` , 因为 `font-size` 直接继承了父元素的大小, 所以 `em` 的误解也就一直存在了.