# 探索

## [Web Animations API](https://developer.mozilla.org/en-US/docs/Web/API/Web_Animations_API)

新的 [Web Animations API](https://developer.mozilla.org/en-US/docs/Web/API/Web_Animations_API) 可以使用 JavaScript 来控制动画, 而且保留了 css 提供的硬件加速优化.

[`Element.animate()`](https://developer.mozilla.org/en-US/docs/Web/API/Element/animate)

[`Element.getAnimations()`](https://developer.mozilla.org/en-US/docs/Web/API/Element/getAnimations)

## [offset](https://developer.mozilla.org/en-US/docs/Web/CSS/offset)

css 属性 `offset` 是一个简写属性用于让元素沿着给定的路径执行动画.

## [prefers-reduced-motion](https://developer.mozilla.org/en-US/docs/Web/CSS/@media/prefers-reduced-motion)

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

