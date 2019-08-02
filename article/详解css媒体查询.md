# 简介

媒体查询(Media Queries)早在在css2时代就存在,经过css3的洗礼后变得更加强大bootstrap的响应式特性就是从此而来的.

简单的来讲媒体查询是一种用于修饰css何时起作用的语法.

> Media Queries 的引入，其作用就是允许添加表达式用以确定媒体的环境情况，以此来应用不同的样式表。换句话说，其允许我们在不改变内容的情况下，改变页面的布局以精确适应不同的设备。(1)

既然媒体查询是用于控制样式的,而样式的使用无外乎如下几种规则:
 - 使用`link`引入
 - 使用`style`标签
 - 使用`style`属性
 - 使用`@import`引入

而显式的使用媒体查询声明样式我们有如下三种方法:
 - 使用`link`引入时使用`media`属性
 - 使用`style`标签时添加`media`属性
 - 在样式中使用条件规则组

我们先来看看`link`的使用方式:

link标签使用媒体查询后基本的样子如下(1):
```
<link rel="stylesheet" type="text/css" href="swordair.css" media="screen and (min-width: 400px)">
```
__一旦使用了媒体查询修饰link标签后,就意味着符合媒体查询后这个样式就会被启用,同样的规则适用于style标签.__

## 例子的解释

那么对于上面的那一句`media="screen and (min-width: 400px)"`就可以解释为:
__当屏幕的宽度大于等于400px的时候应用这条样式规则.__

## 媒体查询的三个部分

上面的例子中我们可以看到多出了一个`media`属性,而`media`中内容就是媒体查询的语法,可以被如下解释:

> 一个媒体查询由一个可选的媒体类型和零个或多个使用媒体功能的限制了样式表范围的表达式组成，例如宽度、高度和颜色。媒体查询，添加自CSS3，允许内容的呈现针对一个特定范围的输出设备而进行裁剪，而不必改变内容本身。(2)

看起来很复杂,但是实际上一个媒体查询的**声明**就分为以下三个部分:
 - 媒体类型 - 形容设备
 - 媒体特性(媒体特征/媒体功能) - 形容设备的状态
 - 逻辑操作符 - 连接多个规则

那么使用上方的例子来说`media="screen and (min-width: 400px)"`中`screen`就是**媒体类型**,
而后面的`and`被称作**逻辑操作符**,
`(min-width: 400px)`则被称作**媒体特性**.

## max 和 min 的基本含义

max 指的是不大于，例如 `max-width:1200px` = `不大于1200px的时候使用该规则中的样式`

min 指的是不小于, 例如 `min-width:1200px` = `不小于1200px的时候使用规则中的样式`

## 媒体类型一览

上文说道媒体查询在css2中就已经有了,所以有很多媒体类型是在css2时代提出的,其中就只有`screen`和`all`被广泛的使用,有很多都被删除掉了.

- 常使用的媒体类型css2制定
  - screen 主要适用于彩色的电脑屏幕
  - all 适用于所有设备 (媒体类型默认值)

- 不常使用的媒体类型
  - print
  - speech

- css2.1被废弃掉的媒体类型(3)
  - tty
  - tv
  - projection
  - handheld
  - braille
  - embossed
  - aura

## 常用的媒体特性

| 名称   | 特性     |
| ------ | -------- |
| width  | 可视宽度 |
| height | 可视高度 |

## 媒体特性完整列表

媒体特性一览:
> https://developer.mozilla.org/zh-CN/docs/Web/CSS/@media#媒体特性

## 媒体查询声明的详细规则

大家可以运行一下这个例子来感受一下:
```html
<!DOCTYPE html>
<html>
<head>
	<meta charset="utf-8">
	<meta name="viewport" content="width=device-width, initial-scale=1" />
	<style type="text/css">
		html,body{
			height: 100%;
		}

		body{
			background-color: aqua;
		}
	</style>
	<style media="screen and (min-width: 400px)">
		body{
			background-color: #000;
		}
	</style>
	<title>test</title>
</head>
<body>

</body>
</html>
```
在这个例子中屏幕宽度大于400像素的时候`body`的背景颜色是黑色,但是一旦低于400像素后就成为了青绿色.

一个媒体查询声明中可以由多个媒体查询组成(使用逗号分割),一个单独的规则是由如下的格式组成的:
| 类型       | 数量    | 默认值 |
| ---------- | ------- | ------ |
| 媒体类型   | 0 / 1   | all    |
| 媒体特性   | n(n!=0) |        |
| 逻辑操作符 | n-1     |        |

也就是说一个媒体查询中可以存在多条规则,对于一个规则需要一个**媒体类型**(默认all)和n个媒体特性(可选),他们之间的连接使用逻辑操作符来连接.

当不填写媒体类型对应的默认规则:
 - (max-width:400px) = all and (max-width:400px)
 - (max-width:400px) and (min-width:200px) = all and (max-width:400px) and (min-width:200px)
 - (max-width:400px) , (min-width:200px) = all and (max-width:400px) , all and (min-width:200px)

## 媒体特性前缀

上面的例子的媒体查询有如下内容`screen and (width: 400px)`如果你看过媒体特性一览表就会发现`min-`这个内容是没有提到的.

大部分媒体特性都是有前缀的,媒体特性前缀主要用于约束媒体特性的作用范围.

- max-xxx 小于指定的最大值返回true
- min-xxx 大于指定的最小值返回true

## 逻辑操作符

所谓的逻辑操作符说白了就是编程中的逻辑操作符,用于连接多个媒体特性表达式.

显示的逻辑操作符一共有两个:
- not 对于匹配到的媒体查询取反
- and 只有连接的两个规则都成立的时候才返回true

**注意**:默认使用逗号分割的多个媒体查询就是or的写法,也就是说逗号就相当于or操作符

特殊的有一个:
- only 不支持更加高级的媒体类型的浏览器检测到only修饰的时候就会抛弃这个规则

__实际使用中然并卵的功能__

## 具体例子及解释

例子1:
```
screen and (min-width: 400px)
```
宽度大于400像素的设备使用这个样式.

例子2:
```
(min-width: 700px) and (orientation: landscape)
```
宽度大于700像素且屏幕为横屏的时候使用这个样式.

例子3:
```
handheld and (max-width:20em), screen and (max-width:30em)
```
表示此CSS被应用于宽度小于20em的手持，或者宽度小于30em的屏幕.


## 条件规则组

所谓的条件规则组就是值媒体的声明不在`link`标签和`style`标签上,而是在css代码中,利用条件规则组我们可以将一块css代码在符合媒体查询的时候应用.

使用方式(BootStrap中的样式代码)
```css
@media (min-width:768px) {
	.lead {
		font-size: 21px
	}
}
```

# 优先级

## 属性 vs 样式

在这个例子中:

```
<!DOCTYPE html>
<html>
<head>
	<meta charset="utf-8">
	<meta name="viewport" content="width=device-width, initial-scale=1" />
	<style type="text/css" media="screen and (min-width: 400px)">
		html,body{
			height: 100%;
		}

		body{
			background-color: aqua;
		}

		@media screen and (min-width: 400px){
			body{
				background-color: #000;
			}
		}
	</style>
	<title>test</title>
</head>
<body>
	
</body>
</html>
```

在`style`标签上声明的属性和在内部的`条件规则组`媒体查询设计的一致,但是内部的`条件规则组`覆盖掉了外部`style`上的媒体查询.

可以看到他们实际上它们之间没有优先级,只有先后执行的顺序,后执行的规则会覆盖掉前面的规则.

## css 中的优先级

考虑如下含有媒体查询的 css 样式：

```css
body {
  background-color: grey;
}

@media screen and (max-width:960px) {
  body {
  background-color: red;
  }
}
@media screen and (max-width:768px) {
  body {
  background-color: orange;
  }
}
@media screen and (max-width:550px) {
  body {
  background-color: yellow;
  }
}

@media screen and (max-width:320px) {
  body {
  background-color: green;
  }
}
```

这些媒体查询的效果就是当页面宽度超过 960px 的时候显示灰色，随着屏幕宽度的减小样式随之改变，使用适合当前页面最大的媒体查询。

默认没有媒体查询的样式就相当于设置了 `max-width:+∞` 虽然实际的媒体查询中没有无限大这个属性。

媒体查询从高到低的排列实际上是符合 css 设计的，即：

> 一个元素匹配多个样式, 只有最后一个样式会被应用到元素身上, 或者说它覆盖了前面的符合条件的样式.

如果页面宽度为 300px 则 `max-width:320px` 可以使用，根据规则 `max-width:550px` 也是符合查询条件的，之所以页面在 300px 宽度的时候没有匹配到 `max-width:550px` 是因为同样符合匹配条件的 `max-width:320px` 更加靠后。

如果没有使用媒体查询的样式放置到样式最后，无论页面如何缩放 body 始终是灰色，因为它匹配所有的宽度的同时还覆盖了前面的所有含有媒体查询的条件。

所以在 css 中使用媒体查询的优先级就是：

> 没有使用查询的放在最前面，含有媒体查询的样式从高到低降序排列

## link 中的优先级

将上例中的内容进行修改，转为 link 标签，变成如下内容：

```html
<link rel="stylesheet" href="./css/960.css" media="screen and (max-width:960px)">
  <link rel="stylesheet" href="./css/768.css" media="screen and (max-width:768px)">
  <link rel="stylesheet" href="./css/550.css" media="screen and (max-width:550px)">
  <link rel="stylesheet" href="./css/320.css" media="screen and (max-width:320px)">
```

实际上这种方式的作用效果和在 css 书写的没有区别，因为 css 规则的最终应用就是 HTML 中的书写的顺序，而这个顺序和 css 中一致所以效果也就一致了。

不过使用这种方式会造成浏览器加载多个 css 文件，这会增加请求负担。

而且浏览器也不会根据媒体查询来动态的加载样式，它只是一股脑的将所有的样式引入。

# 引用&参考


(1)

> http://www.swordair.com/blog/2010/08/431

(2)

> https://developer.mozilla.org/zh-CN/docs/Web/Guide/CSS/Media_queries

(3)

> https://developer.mozilla.org/zh-CN/docs/Web/CSS/@media

> https://developer.mozilla.org/zh-CN/docs/Web/CSS/At-rule#条件规则组

> https://www.zhangxinxu.com/wordpress/2011/08/css3-media-queries%E7%9A%84%E4%BA%9B%E9%87%8E%E5%8F%B2%E5%A4%96%E4%BC%A0/

# 额外补充

更多的详细的例子:
> http://www.cnblogs.com/lguow/p/9316598.html

使用媒体查询注意的常见错误:
> https://blog.csdn.net/qq_37558379/article/details/80865331

电脑分辨率对应的媒体查询:
> https://blog.csdn.net/happydecai/article/details/81013319

# 暗坑

在写例子的时候我使用到了两个浏览器最新的firefox和最新的chrome,有趣的事情是二者在`style`标签上使用`media`属性表现不同.

firefox中不写`<meta name="viewport" content="width=device-width, initial-scale=1" />`也是正常运行,但是chrome就不可以.