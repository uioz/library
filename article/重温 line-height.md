# 标题: 重温 line-height

我个人对于 css 中某些作用比较复杂的属性没有做过太多深入的了解例如本篇文章提到的 `line-height`.  
知名前端博主张鑫旭很久前上传了有关 css 几个基础属性的讲解视频, 其中包括 `line-height` 属性.  
这篇文章实际上是该视频的笔记.  

视频地址:
> https://www.imooc.com/learn/403

# line-height 高度计算方式

两行文字的 baseline 的距离就是 line-height 的高度.

## 基线存在的意义

我们利用行高来确定两个基线之间的距离, 从而确定基线的位置.  
知道了基线的位置, 字体根据基线来进行渲染.  

**注意**: 不同字体的 baseline 所在的位置不同.

# 行内框模型

## 内容区域

内容区域大小由 `font-size` 和字体决定.  
换句话说字体大小不一定等于内容区域的大小(实际上大部分都不等于).  
不过在 "simsun" 字体下, `font-size` 大小正好等于内容区域的大小.

## 内联盒子

内联盒子会让内容拍成一行显示, `display` 显示值以 `inline` 显示的元素, 以及空白的文字节点都是内联盒子被称为 "匿名内联盒子".

## 行框盒子

行框盒子由内联盒子组成, 并且控制折行显示

## 包含盒子

包含盒子由一行行的行框盒子组成.

# line-height 机制

行高度的表现不是行高, 而是内容区域和行间距控制的.  
只不过正好 内容区域高度 + 行间距 = 行高.

所以行间距的计算公式就等于 行高 - 内容区域高度 = 行间距.  
不同字体的 `font-size` 值和其对应的内容区域大小不一致, 计算时候需要获取其实际内容区域值计算而不是直接使用字体大小.  

## 半行间距

行间距是存在于内容区域的上下两端的, 而半行间距就是内容区域上面或者下面的行间距这个距离被称为 "半行间距".  
半行间距是相等的所以 行间距 = 半行间距*2

# line-height 单位

line-height 可以使用多种单位, 这其中包括:  
- normal
  - 跟着用户的浏览器走, 这意味着不同的浏览器不同. 另外不同的元素和不同的字体都会影响到 line-height 的值.
- <number>
  - 根据当前 font-size 进行计算. 20px 的 font-size 对应的 line-height 是 30px.
- <length>
  - 使用具体值来表示行高值. 例如 1.3em 1.5rem 10px 20pt.
- <percent>
  - 相对于设置了该 line-height 属性的元素的 font-size 大小进行计算.
- inherit
  - 继承父元素的 line-height

## number/length/precent 的使用区别

**注意**:line-height具有继承特性.

line-height:1.5 表示让所有的后代元素的 line-height 都调整为相对于自身的 font-size 进行计算.  
line-hegiht:1.5em/1.5rem/150% 表示让所有元素获取当前元素的 line-height 的高度.  

# line-height 与图片

有如下的代码:
```html
<p><img src=""></p>
```
比较常见的一个现象就是 img 下方有额外的空白, 造成这种情况的主要原因有两个:
1. inline 元素的默认垂直对其方式是 baseline 一般是英文字母的底边缘.
2. 元素内部默认有两个空白文本节点伴随在 inline 元素的前后

也就是说原来的:
```html
<p><img src=""></p>
```
相当于:
```html
<p>伪装的空白文本节点<img src="">伪装的空白文本节点</p>
```
所以图片的下边缘会和文字的 baseline 对齐, 导致高度提升.

## 解决方式

1.图片块状化
```css
img {display:block;}
```
因为空白文本节点只会对 inline 和 inline-block 其作用, 只要将其修改为 block 就可以避免这种问题.

2.修改为底线对齐
```css
img{vertical-align:bottom;}
```

3.行高足够小
```css
.box{line-height:0}
```
当行高变小的时候 baseline 会上移, 此时图片的底边缘就会比 baseline 还要靠下此时间距也就消失了.

# line-height 实际应用

## 图片水平垂直居中

基本实现思路, 我们将图片视为一个文字(因为两者表现起来都是 "行内元素"), 那么文字如何水平垂直居中图片也就是相同的手段:
```html
<div><img src=""></div>
```
```css
div{
  line-height:300px;
  text-align:center;
}

div > img{
  vertical-align:middle;
}
```

## 多行文本水平垂直居中

思路和上方一致, 我们将多行文本视为一个文字, 然后让他水平垂直居中, 所以我们需要将这些多行文本使用一个`<span>`来将这些文本包裹起来:
```html
<div class="box"><span class="text">一长段文字,请酌情添加内容.</span></div>
```
```css
.box{
  line-height:250px;
  text-align:center;
  background:#eee;
}

.box > .text{
  display:inline-block;
  background:#fff;
  text-align:left;
  line-height:normal;
  vertical-align:middle;
}
```