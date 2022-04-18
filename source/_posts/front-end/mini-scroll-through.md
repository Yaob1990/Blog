---
title: 小程序中的滚动穿透
date: 2022-04-18 22:01:25
categories: 前端
tags:
  - 小程序
img: ../../coverImages/zhifubao-mini.png
---

在小程序开发中，弹出层滚动穿透是个比较棘手的问题。如下图，蓝色部分滚动，底部也跟着一起滚动，就是滚动穿透。

常见的处理方法比如，禁止滚动，并不生效。而给主体加 `overflow: hidden` 又会导致主体滚动条高度为 0，需要关闭时候记录滚动条位置，体验也不好。

![](/images/2022/04/18/16502908272281.jpg)

## touch-action


| 值    | 描述             |
|-------|------------------|
| auto  | 启用             |
| none  | 禁用             |
| pan-x | 启用单指水平移动 |
| pan-y | 启用垂直手势 |
| manipulation | 浏览器只允许进行滚动和持续缩放操作。任何其它被auto值支持的行为不被支持。启用平移和缩小缩放手势，但禁用其他非标准手势，例如双击以进行缩放。 禁用双击可缩放功能可减少浏览器在用户点击屏幕时延迟生成点击事件的需要。 |
| pinch-zoom | 启用多手指平移和缩放页面 |
| pan-left, pan-right,pan-up,pan-down | 启用指定方向滚动开始的单指手势 |

在 mask 和 popup 元素上面禁用即可：

```css
.mask, .popup {
    touch-action: none
}
```

这里和 overflow 方式类似，都不是特别好的办法，能解决部分问题，但是不完美。

**注意**：这里适合 popup 内部没有滚动的情况

## 禁止冒泡和默认行为

preventDefault：阻止默认滚动动作的执行。
stopPropagation: 阻止冒泡，阻止事件由下向上传递。

实现上，小程序这里需要借助 sjs 的能力来实现：
sjs 文件：
```js
<!-- sjs 文件 -->
function disableScroll(e){
  e.preventDefault()
}

function enableScroll(e){
// 阻止冒泡
  e.stopPropagation()
}

export default {
  disableScroll,
  enableScroll
}
```

axml文件：

```html
<import-sjs from="./index.sjs" name="sjs" />

<view class="wrapper" >
  <view a:for="{{[1, 2, 3, 4, 5, 6, 7, 8, 9]}}" a:for-item="i">
    <view class="item">{{i}}</view>
  </view>
  
  <view onTouchMove="{{sjs.disableScroll}}">
    <view class="mask" />
    
    <view class="popup" >
      <scroll-view scroll-y class="popup-scroll" onTouchMove="{{sjs.enableScroll}}">
        <view a:for="{{[1, 2, 3, 4, 5, 6, 7, 8, 9]}}" a:for-item="i">
          <view class="item2">{{i}}</view>
        </view>
      </scroll-view>
    </view>
  </view>
</view>
```

或者，我们把阻止默认事件放到 mask 层上面，那么下面的 scroll-veiw 不处理，也能够正常滚动

```html
<import-sjs from="./index.sjs" name="sjs" />
<view class="wrapper" >
  <view a:for="{{[1, 2, 3, 4, 5, 6, 7, 8, 9]}}" a:for-item="i">
    <view class="item">{{i}}</view>
  </view>
  
  <view >
    <view class="mask" onTouchMove="{{sjs.disableScroll}}" />
    <view class="popup" >
      <scroll-view scroll-y class="popup-scroll" >
        <view a:for="{{[1, 2, 3, 4, 5, 6, 7, 8, 9]}}" a:for-item="i">
          <view class="item2">{{i}}</view>
        </view>
      </scroll-view>
    </view>
  </view>
</view>

```






