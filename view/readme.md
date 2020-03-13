# web 开发优秀资源索引

# JavaScript

[Understanding Scope and Context in JavaScript](http://ryanmorr.com/understanding-scope-and-context-in-javascript/)理解 JavaScript 中的 "上下文" 与 "作用域".

- [冴羽深入JavaScript系列](https://github.com/mqyqingfeng/Blog)

# 浏览器

## 浏览器工作解析

[渲染树构建、布局及绘制](https://developers.google.com/web/fundamentals/performance/critical-rendering-path/render-tree-construction?hl=zh-cn)

## 渲染优化

[浏览器回流重绘详解(英文)](https://www.phpied.com/rendering-repaint-reflowrelayout-restyle/), 解释了回流(reflow) 重绘(repaint) 的基本概念.

[在chrome中加速渲染-层模型](https://www.html5rocks.com/zh/tutorials/speed/layers/), 解释了浏览器绘制(paint) 的过程.

[是什么导致布局与回流](https://gist.github.com/paulirish/5d52fb081b3570c81e3a), 给出了一份可以导致浏览器回流重绘的列表, 以及相关资源.

[css triggers](https://csstriggers.com/), 一个工具网站提供了一份完整的列表, 从多方面展示一个 css 属性的变化是否会影响页面渲染.

## 加载优化

[Loading Performance(英文)](https://developers.google.com/web/fundamentals/performance/get-started), 由 google 开发者平台提供的首屏加载优化指南, 涵盖了几乎所有关于加载优化的基本概念.

[以用户为中心的性能指标](https://developers.google.com/web/fundamentals/performance/user-centric-performance-metrics), 由 google 开发者平台提供的一篇有关页面加载指标的文章, 理解加载指标是衡量优化的前提条件.

## 综合优化

[Vue-SSR 优化方案详细总结](https://zhuanlan.zhihu.com/p/93199714), 一篇有关 Vue-SSR 优化的指南.

## 性能分析

[pageSpeed Insights](https://developers.google.com/speed/pagespeed/insights/), 由 google 提供的一款在线的网页性能分析工具, 可以通过多个指标衡量页面加载速度, 并且给出对应的优化方式.

# 综合性指南

[Google 开发者平台-WEB 基础](https://developers.google.com/web/fundamentals?hl=zh-cn) 提供了大量的优质 web 技术文章, 页面左侧提供了文章索引.

[ES6标准入门-阮一峰](http://es6.ruanyifeng.com/), ES6 入门教程 - ECMAScript 6入门, 非常全面的ES6电子书籍, 页面左侧提供了章节索引.

# CSS 关键知识概念

## CSS Flow Layout (`display: block`, `display: inline`)

- [Block and Inline Layout in Normal Flow](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_Flow_Layout/Block_and_Inline_Layout_in_Normal_Flow)
- [Flow Layout and Overflow](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_Flow_Layout/Flow_Layout_and_Overflow)
- [Flow Layout and Writing Modes](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_Flow_Layout/Flow_Layout_and_Writing_Modes)
- [Formatting Contexts Explained](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_Flow_Layout/Formatting_Contexts_Explained)
- [In Flow and Out of Flow](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_Flow_Layout/In_Flow_and_Out_of_Flow)



[Visual formatting model](https://developer.mozilla.org/en-US/docs/Web/CSS/Visual_formatting_model) 视觉格式化模型, 该文章描述了文档布局背后所运转的机制, 即 "盒" 模型, 以及 "盒" 模型与布局之间的基本原则.

[Layout and the containing block](https://developer.mozilla.org/en-US/docs/Web/CSS/Containing_block) 布局与包含块, 该文章准确的描述了 "包含块" 的概念, 指出了一个元素的大小和位置是如何计算的准则.

[Block formatting context](https://developer.mozilla.org/en-US/docs/Web/Guide/CSS/Block_formatting_context) 块格式化上下文, 该文件探讨了 BFC 的概念, 以及如何触发 BFC, 并且讨论了它与 "浮动" 之间的关系.

[The stacking context](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_Positioning/Understanding_z_index/The_stacking_context) 层叠上下文(堆叠上下文), 该文章阐述了 z-index 具体表现, 以及背后的原理是根据 "层叠上下文" 这一概念运行, 并具体的探讨了 "层叠上下文" 在不同情况下的表现.

[Stacking with floated blocks](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_Positioning/Understanding_z_index/Stacking_and_float) 层叠和浮动块(之间的相互作用), 该文章讨论了当 "定位中使用了 z-index 来控制层叠的时候, 遇到浮动元素该如何渲染" 这以具体的问题.