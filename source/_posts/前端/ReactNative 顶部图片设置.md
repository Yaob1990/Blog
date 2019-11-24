---
title: React Native 沉浸式状态栏和导航栏
date: 2019-11-24 21:57:00
categories: 前端
img: ../../coverImages/reactNative.png
top: true
tags:
  - React Native
  - 混合开发
typora-root-url: ..\..
---



最近开始接触 React Native 的开发工作，对原生开发基本不了解，里面很多大大小小的坑。这次开发内容不是很复杂，数据交互不是非常多，主要是页面的布局。

##  如何实现一个顶部隐藏状态栏并显示导航栏的页面？

![产品图片](/images/image-20191124220426772.png)

## 实现方式

1. 使用`Image`实现图片的显示，这里的图片最好使用` ImageBackground `组件，区别是` ImageBackground `可以嵌套其他组件
2. 导航栏，我一开始是使用绝对定位的方式进行模拟的，但是越想越不对，实现肯定是有问题的，果然在`navigationOptions `中找到了`  headerTransparent   `属性，使用它，导航栏就可以实现透明显示。

3. 状态栏透明显示：

   ```javascript
   StatusBar.setBackgroundColor('transparent');
   StatusBar.setTranslucent(true);
   ```

   

## 可用参考示例

[沉浸式状态栏]( https://github.com/Yaob1990/ReactNativePlayGround/blob/master/src/pages/Bar.js )



### 参考文章

1. [React Native 中的状态栏](https://www.jianshu.com/p/8075ccc84d07)
2. [react native沉浸式(透明)状态栏与标题导航栏](http://www.jeepxie.net/article/558579.html ) 