# 探索

## JavaScript

### [Web Animations API](https://developer.mozilla.org/en-US/docs/Web/API/Web_Animations_API)

新的 [Web Animations API](https://developer.mozilla.org/en-US/docs/Web/API/Web_Animations_API) 可以使用 JavaScript 来控制动画, 而且保留了 css 提供的硬件加速优化.

[`Element.animate()`](https://developer.mozilla.org/en-US/docs/Web/API/Element/animate)

[`Element.getAnimations()`](https://developer.mozilla.org/en-US/docs/Web/API/Element/getAnimations)

### [Performance](https://developer.mozilla.org/en-US/docs/Web/API/Performance)

`Performance` 接口用于访问当前页面相关的性能信息.

另外该接口还被以下接口所扩展, 提供了增强的能力用于提供更加细化的性能信息和接口.

- [Performance Timeline API](https://developer.mozilla.org/en-US/docs/Web/API/Performance_Timeline)
- [Navigation Timing API](https://developer.mozilla.org/en-US/docs/Web/API/Navigation_timing_API)
- [User Timing API](https://developer.mozilla.org/en-US/docs/Web/API/User_Timing_API)
- [Resource Timing API](https://developer.mozilla.org/en-US/docs/Web/API/Resource_Timing_API)

### [Background Tasks API](https://developer.mozilla.org/en-US/docs/Web/API/Background_Tasks_API)

幕后任务接口提供了将排序后的任务交由用户代理在空闲时间自动执行这些任务的能力.

## CSS

### [offset](https://developer.mozilla.org/en-US/docs/Web/CSS/offset)

css 属性 `offset` 是一个简写属性用于让元素沿着给定的路径执行动画.

### [prefers-reduced-motion](https://developer.mozilla.org/en-US/docs/Web/CSS/@media/prefers-reduced-motion)

`prefers-reduced-motion` 媒体查询特性用于探测用户是否请求最小化系统动画或运动.

例如当用户不需要使用动画的时候可以使用媒体查询关闭所有的动画:

```css
@media (perfers-reduced-motion: reduce) {
    *{
        animation-name:none !important;
        transition-property: none !important;
    }
}
```

当执行动画脚本的时候可以通过以下判断决定是否执行动画脚本:

```javascript
// 判断元素是否使用了动画
if(!('animate' in elem)){
  return;
}

// 或者判断媒体查询是否给定
if(matchMedia('(perfers-reduced-motion)').matches){
  return;
}
```

### [CSS Scroll Snap](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_Scroll_Snap)

"滚动捕获" 是 CSS 的一个模块, 它引入 "滚动捕获点" 这个概念, 他会确保滚动行为结束后尽可能的停留在这些 "滚动点" 上.

[CSS Scroll Snap specification](https://drafts.csswg.org/css-scroll-snap-1/)给我们了一种途径, 当用户滚动文档时候 "捕获" 滚动到一个确定的位置. 这有助于在移动设备以及某些桌面应用上创建像原生应用程序样的体验.

### [CSS Logical Properties and Values](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_Logical_Properties)

**CSS 逻辑属性** 是 CSS 的一个模块, 引入了 "逻辑属性" 提供了使用逻辑值而不是物理值来控制布局的方向以及尺寸.

PS: 这个规范可能是 CSS 最大的 "补丁" 修复了以前 CSS 关于布局规范中的很多漏洞. 新规范基本覆盖了所有传统布局, 在传统布局上提供了使用 "逻辑属性" 的运作方式.