# 分析

使用了常见的 `text-shadow` 叠加的效果制作出霓虹灯的光晕效果, 提前制定颜色后再结合 `hub-rotate` 来实现颜色过渡时机上是一个比较取巧的做法.

再动画的时间线上还是做了一些细节处理的, 为了实现更加拟真的效果, 动画时间线并没有严格按照 "程序式" 思考方式, 而是更加贴近现实在 `0% 30%,70% 100%` 实现了帧动画, 且在 `30%, 70%` 处减弱了光晕的效果, 模拟更加真实的霓虹.

在本例中最核心的代码是通过设置 `animation-delay` 来控制每个元素执行动画的时机来模拟霓虹灯律动的效果, 使用了 "自定义css属性" 技巧来简化了代码的编写:
```html
<span style="--i: 1">h</span>
<span style="--i: 2">e</span>
<span style="--i: 3">l</span>
<span style="--i: 4">l</span>
<span style="--i: 5">o</span>
<span style="--i: 6">w</span>
<span style="--i: 7">o</span>
<span style="--i: 8">r</span>
<span style="--i: 9">l</span>
<span style="--i: 10">d</span>
<span style="--i: 11">!</span>
```
```css
span {
  animation-delay: calc(0.1s * var(--i));
}
```