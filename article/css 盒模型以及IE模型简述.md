# 简介

这篇文章中只讨论盒模型, 布局不在讨论的范围内.

# css 盒模型 level3(部分)

## 简介

CSS描述了在将文档树转为一组盒子的时候源文档中每一个元素以及文本该如何摆放, 盒子的大小, 位置, 以及在画布上的层叠级别都是取决于它的 css 属性.

每个 css 盒子都有一个矩形的内容区域, 以及一个围绕着内容的内边距带, 围绕着内边距的边框, 和最外边围绕着边框的外边距. 而 [大小属性](https://www.w3.org/TR/css-sizing-3/#sizing-property), 和可以其他的控制布局属性来定义内容区域的大小. 盒子外观属性 padding border margin 和他们的完整写法定义了其他区域的大小. 外边距和内边距在本模块中定义, 边框在  [[css-backgrounds-3\]](https://www.w3.org/TR/2020/WD-css-box-3-20200421/#biblio-css-backgrounds-3) 中定义.

## css 盒模型

每一个盒子都有一个内容区域(可以包含文本后代盒子以及图片或者其他可替换元素 等)，以及可选的围绕着它的 padding , border , margin 区域. 不同区域的大小，是由各自对应的属性所控制. 这些属性可以被设置为0(对于 margin 来说可以为负值). 下列图示展示了技术名词与盒子不同部分的对应关系.

![Diagram of a typical box, showing the 		content, padding, border and margin areas](https://www.w3.org/TR/2020/WD-css-box-3-20200421/images/box.png) 

margin border padding可以被拆分为Top right bottom left 四个分段, 每一个分段由对应的属性所控制. 这4个区域的周长呢, 被称为边缘, 然后每一个边缘可以被拆分为Top right bottom left 四个边. 所以每个盒子呢都有4个边缘 ,每个边缘由4个边构成:

- 内容边缘或者内部边缘

内容边缘围绕着一个由盒子所构成的矩形, 被盒子的 width 和 height 所控制, 而盒子的大小通常取决于元素的内容或者它对应包含块的大小. 内容边缘的四条边共同定义了盒子的内容盒子.

- 内边距边缘

内边距边缘围绕着盒子的内边距，如果内边距有一边的宽度为0, 那么内边距边缘的这条边将会和内容区域边缘相同的一边重合. 内边距边缘的4条边共同定义了盒子的内边距盒子, 内边距盒子包含了内容, 以及内边距区域.

- 边框边缘

边框边缘围绕着盒子的边框，如果边框的某一边的宽度为0，那么该边框边缘的一侧将会和内边距边缘重合. 边框边缘的4条边共同定义了盒子的边框盒子，边框盒子包含盒子的内容, 内边距, 以及边框区域.

- 外边距或外部边缘

外边距边缘围绕着盒子的外边距. 如果外边距的某一边的值为0，那么这一边的外边距边缘将会和边框边缘所重合，外边距的4个边共同定义了盒子的外边距盒子, 它可以包含盒子的内容, 内边距, 边框, 以及外边距区域.

盒子的内容, 内边距, 以及边框区域的背景, 由 background 属性控制. 边框区域的绘制可以额外的通过边框相关属性控制. 外边距总是透明的.

When a box [fragments](https://www.w3.org/TR/css-break-3/#fragmentation-model)—is broken, as across lines or across pages, into separate [box fragments](https://www.w3.org/TR/css-break-4/#box-fragment)—each of its boxes ([content box](https://www.w3.org/TR/2020/WD-css-box-3-20200421/#content-box), [padding box](https://www.w3.org/TR/2020/WD-css-box-3-20200421/#padding-box), [border box](https://www.w3.org/TR/2020/WD-css-box-3-20200421/#border-box), [margin box](https://www.w3.org/TR/2020/WD-css-box-3-20200421/#margin-box)) also fragments. How the content/padding/border/margin areas react to fragmentation is specified in [[css-break-3\]](https://www.w3.org/TR/2020/WD-css-box-3-20200421/#biblio-css-break-3) and controlled by the [box-decoration-break](https://www.w3.org/TR/css-break-4/#propdef-box-decoration-break) property.

> https://www.w3.org/TR/2020/WD-css-box-3-20200421/

# IE 模型

IE 浏览器没有官方的文档可以进行参考, 这里参考了国外的几篇文章以及 wiki:

> https://www.456bereastreet.com/archive/200612/internet_explorer_and_the_css_box_model/
>
> https://www.jefftk.com/p/the-revenge-of-the-ie-box-model
>
> https://en.wikipedia.org/wiki/CSS_box_model

在1996年的12月份 w3c 制定了第一份 css 标准, 在规范中它使用了这张图来描述盒模型:

![img](https://upload.wikimedia.org/wikipedia/commons/thumb/6/64/W3C_and_Internet_Explorer_box_models.svg/300px-W3C_and_Internet_Explorer_box_models.svg.png)

当时的微软发布了支持 css1 规范的 [Trident](http://en.wikipedia.org/wiki/Trident_(layout_engine)) 布局引擎, 而该引擎的和盒模型就像 css 1 如上的样子.

简单来说就是在 IE 模型中, width 包含了 内容区域 + 内边距 + 边框 这三者的大小而不是只控制内容区域.

用今天的话来说相当全局使用了 `box-sizing: border-box` .

IE 5.5 之前的所有的 IE 浏览器默认使用 IE 和模型. 许多人似乎不知道的是，[在标准兼容模式下](http://msdn.microsoft.com/workshop/author/dhtml/reference/objects/doctype.asp)，IE 6和更高版本使用W3C盒模型。这是一件好事，因为这意味着仅当使用使IE使用标准兼容模式的DOCTYPE时，问题才会出现在IE / Win 5.5及更低版本中。

但是出于向后兼容的原因，默认情况下，所有版本在默认情况下仍以通常的非标准方式运行（请参考 "[怪异模式](https://en.wikipedia.org/wiki/Quirks_mode)"）. Mac 上的 IE 不受此行为影响.

## 解决方式

IE 已经不再是主流浏览器, 这部分便不再翻译了, 详情可以参考这篇文章给出了数中解决方法:

> https://www.456bereastreet.com/archive/200612/internet_explorer_and_the_css_box_model



