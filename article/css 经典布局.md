# 圣杯布局

> https://www.jianshu.com/p/81ef7e7094e8

## 概述

圣杯布局用于是创建三列布局的一种手段.

适合两侧的列宽度固定, 中间的列宽度自适应.

## HTML

```html
<div id="header">header</div>
  <div id="container">
    <div id="center" class="column">Lorem ipsum dolor sit amet, consectetur adipisicing elit. Voluptate, rem modi ducimus neque. Soluta minus repellendus similique aliquam voluptates facilis vitae expedita reiciendis, sequi reprehenderit itaque, sint autem vel, voluptatem!</div>
    <div id="left" class="column"></div>
    <div id="right" class="column"></div>
  </div>
  <div id="footer">footer</div>
```

## CSS

```css
body{
  min-width:550px;
}
#header,#footer{
  background:#999;
  height:100px;
  line-height:100px;
  color:#fff;
  text-align:center;
}

#container{
  padding: 0 150px 0 200px;
  height:200px;
  background:rgba(255,0,0,.2);
}

#container > .column {
  height:inherit;
  float: left;
}

#center {
  background:green;
  width: 100%;
}

#left {
  background:red;
  width: 200px; 
  margin-left:-100%;
  position:relative;
  right:200px;
}

#right {
  background:blue;
  width: 150px; 
  margin-right:-150px;
}

#footer {
  clear: both;
}
```

## 额外提示

`margin-xxx` 包括 `margin` 属性使用百分比值的时候, 该值的大小是相对于父元素的 `width` 进行计算的.

这也是 `margin-left:-100%` 可以移动一个父元素大小的原因.

# 双飞翼布局

## 概述

基本编写方式和圣杯布局一致, 但是可以不使用绝对定位.

## HTML

```html
  <div id="header">header</div>
  <div id="container" class="column">
    <div id="center">Lorem ipsum dolor sit amet, consectetur adipisicing elit. Accusamus excepturi vel modi soluta mollitia? Porro dolor tempore assumenda aliquid cupiditate fuga suscipit adipisci ab officia, delectus cum, eum unde, cumque.</div>
  </div>
  <div id="left" class="column"></div>
  <div id="right" class="column"></div>
  <div id="footer">footer</div>
```

## css

```css
body{
  height:200px;
}

#header,#footer{
  background:#999;
  height:100px;
  line-height:100px;
  color:#fff;
  text-align:center;
}

#container{
  width:100%;
}

.column{
  height:inherit;
  float:left;
}

#center{
  background:green;
  height:inherit;
  margin-left: 200px;
  margin-right: 150px;
}

#left{
  background:red;
  width: 200px;
  margin-left:-100%;
}

#right{
  background:blue;
  width: 150px;
  margin-left:-150px
}

#footer{
  clear:both;
}

```





