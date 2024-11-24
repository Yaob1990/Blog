---
title: 从 Tailwind 看 css 中的原子化流变
categories:
  - front-end
img: ../../coverImages/tailwindcss.png
tags:
  - css
date: 2020-12-14 23:00:00
---

> 回不到的过去，忘不掉的人。某个路口，转角再次遇见，她还是她吗，你还是你吗？

## 禅意花园
[禅意花园](http://www.csszengarden.com/tr/zh-cn/)，做前端的人或多或少都听说过项目。一样的 HTML 结构，用不同的 css 去装饰，形成截然不同的页面风格，探索 css 之美。
`html css` 这一对欢喜冤家，谁是主，谁是客，一直没有定论。

## 关注点即世界

```html
<div class='btn btn-primary'> // css 依附于 html
<div class='w50 h50 black mt30'>   // css 原子化
```
### html-friendly
第一行，使用`class`修饰`html`，也就是html-friendly，样式服务于结构。`class`每一个是有多个`css`属性，这时候的代码也更接近于日常工作中的代码，但是这样的代码如果调整起来，我们经常是直接在现有的`class`上面修改，代码会越来越长，重复的逻辑也越来越多，如果项目一直在维护，那么css基本上会一直增大。毕竟，绝大多数时候，我们只在意功能的实现。

### css-friendly
第二行代码,是上古时期 css 原子化的代表，推崇样式的复用。`width:50px;height:50px;color:balck;margin-top:30px`。每一个`class`都代表着一种 css 属性。原子化最早是雅虎提出来的，代表作是ACSS。ACSS 表示的是原子化 CSS（Atomic CSS），是 Yahoo 提出来的一种独特的 CSS 代码组织方式，应用在 Yahoo 首页和其他产品中。ACSS 的独特性在于它的理念与一般开发人员的理解有很大的不同，并挑战了传统意义上编写 CSS 的最佳实践，也就是关注点分离原则。ACSS 认为关注点分离原则会导致冗余、繁琐和难以维护的 CSS 代码。

ACSS 的原则是把 CSS 样式打散成尽可能小的部分，每个 CSS 类只对应一条样式规则，从而达到最大化的可复用性。比如 CSS 类 M(10px)所对应的样式规则是 margin: 10px。

岁月的洗礼，原子化，似乎已经进入了历史的垃圾堆，不管是工作中还是开源框架里，很少能看到大规模使用原子化的项目（ACSS github 1.1k start），直到 tailwind 的诞生。

## 要有光
原子化虽然用的很少了，但是他的影子一直都在，项目里面`pl10 mt5`这种魔法写法一直都在。
现代化的前端开发，追求自适应，语义化，这些都不是传统原子化能够承担的。
[tailwindcss](https://tailwindcss.com/)，应运而生，扛起了 css 原子化的大旗。
主要优点：
* 着眼于现代化前端，默认单位是 `rem` 而不是 `px`
* 丰富的预设，预设字体大小，丰富的颜色
* 响应式设计，提供一系列断点判断
* 方便扩展，根据你的项目需要，自定义相关的原子类
* 兼容各大框架，并有详细说明

## 是骡子是马
[欧姆龙血压计数据分析](https://github.com/Yaob1990/OMRON_Blood_Pressure_Analyse)，使用了`tailwindcss`作为基础。
![pc](/images/16079566659385.jpg)

![mobile](/images/16079566841086.jpg)

头部的响应式效果，下面是这部分代码：

```html
<div class="relative h-96">
    <img class="w-full h-96 object-cover absolute" src="../assets/bg.jpg" />
    <div class="w-full h-96 bg-black absolute opacity-40"></div>
    <div
      class="w-full h-96 absolute flex flex-col  justify-center items-center"
    >
      <div class="font text-white text-center text-4xl  font-bold">
        欧姆龙血压计数据(CSV)解析工具
      </div>
      <Button class="mt-8" @click="upload">开始解析数据</Button>
    </div>
  </div>
```

瞥一眼`w-full h-96 absolute flex flex-col justify-center items-center` 这一大堆都是什么鬼东西...
```css
width: 100%;
height: 96rem;
position: absolute;
display: flex;
flex-direction: column;
justify-content: center;
align-items: center;
```
如果你觉得这样写还是有点难看，可以这样写：
```
<div class='wrapper'>
.wrapper{
    @apply w-full h-96 absolute flex flex-col justify-center items-center
}
```
是不是感觉好那么一点点...几个class，完成了响应式布局，感觉还是很不错的，而且不用想一堆 class 命名。

## 真的那么好？
初见总是美好的，相处总是困难的。

* 写px单位，要么换算成 rem，要么自定义属性
* 需要新学一堆 class 语法
* 要思考，对公共类进行抽象，定义，不能无脑写
* 对 css 技能要求高，在 一堆的原子中，写一个用不到，或者不生效的 class 是不能被容忍的
* 项目维护是个灾难，如果接手的人不熟悉这套模式，他会想你个傻x...

个人项目，天马行空，我的地盘我做主，Tailwind 用起来，小巧可人，甜而不腻。
公司项目，代码冗余并不是不能接收的事情，考虑到项目维护，暂不推荐。

