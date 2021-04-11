---
title: 前端 font 优化
date: 2021-04-11 16:00:00
categories: 前端
img: ../../coverImages/font.jpeg
tags:
  - css
---

开发中，一般直接引入字体，使用即可，似乎字体本身并没有很多可以说道的地方，其实不然。字体本身涉及到印刷工业，是一个历史悠久的行业，css 字体的大部分属性都可以在印刷工业中找到映射。这篇博客，整理常见的字体使用细节。

## 衬线体和非衬线体
衬线体（英语：Serif）是一种有衬线的字体，又称为有衬线体、衬线字、曲线描边字，俗称白体字；而与之相对的，没有衬线的字体则被称为无衬线体。衬线是字形笔画的起始段与末端的装饰细节部分。
一般的 web 开发都倾向于无衬线体，我们在定义字体时候，应该设置完整的字体属性，衬线体或者非衬线体结尾。

```css
// 谷歌的字体设置
p {
   font-family: 'Roboto',arial,sans-serif;
}
```

![](images/16181269068142.jpg))

## 字重
### 字重和字体名称的关系
我们经常在设计稿重看到 `PingFangSC-Regular`、`PingFangSC-Medium`、`PingFangSC-Semibold` 这些字体，是3个字体吗，那么这些字体在css中应该怎么写？当成独立字体直接写，好像也没有问题，也可以解析。

但是这样，似乎是有点问题的，这些字体属于平方字体，只不过是不同的字重而已。我们直接写 `PingFangSC` 然后设置字重就可以了。

如果只是写 `PingFangSC-Medium`，不设置字重，其实也是无法生效的，会显示默认字重，也就是 `PingFangSC-Regular`。

进一步，我们为什么需要写 `PingFangSC` 字体，这个字体是苹果系统默认的中文字体。所以，设计稿是平方字体的时候，直接写 `font-weight` 好了。

### 字体包的使用

> NotoSansSC 字体
> NotoSansSC-Thin.otf
> NotoSansSC-Light.otf
> NotoSansSC-Regular.otf
> NotoSansSC-Medium.otf
> NotoSansSC-Bold.otf
> NotoSansSC-Black.otf

现代字体往往会提供多种自重，相对于以往的 css bolder 会更加的精细，写法如下：

```css
@font-face {
    font-family: 'Noto Sans SC';
    font-style: normal;
    font-weight: 100;
    src:'./NotoSansSC-Thin.otf';
}

@font-face {
    font-family: 'Noto Sans SC';
    font-style: normal;
    font-weight: 300;
    src:'./NotoSansSC-Light.otf';
}

@font-face {
    font-family: 'Noto Sans SC';
    font-style: normal;
    font-weight: 400;
    src:'./NotoSansSC-Regular.otf';
}

@font-face {
    font-family: 'Noto Sans SC';
    font-style: normal;
    font-weight: 500;
    src:'./NotoSansSC-Medium.otf';
}

@font-face {
    font-family: 'Noto Sans SC';
    font-style: normal;
    font-weight: 700;
    src:'./NotoSansSC-Bold.otf';
}

@font-face {
    font-family: 'Noto Sans SC';
    font-style: normal;
    font-weight: 900;
    src:'./NotoSansSC-Black.otf';
}
```

使用：

```css
p {
    font-family: 'Noto Sans SC', sans-serif;
    font-weight: 900;
}
```

### 常见计稿字体对应字重font-weight

* 100 - Thin
* 200 - Extra Light (Ultra Light)
* 300 - Light
* 400 - Regular (Normal、Book、Roman)
* 500 - Medium
* 600 - Semi Bold (Demi Bold)
* 700 - Bold
* 800 - Extra Bold (Ultra Bold)
* 900 - Black (Heavy)

## 字体裁切
因为中文字体都比较大，一般都需要使用webfont等方式缩小字体大小。

### 字蛛
字蛛的使用限制：
1. 静态渲染内容
2. 字体必须是 ttf 类型

字体可以理解成是一种图片，裁切，保留需要用到的字符。
使用比较简单，把需要用到的中文写到 html 中，运行字蛛的命令即可得到需要的字符。

### google font
谷歌字体的引用地址是可以使用的：[使用地址](https://fonts.gstatic.com/)
![](/images/16181293778639.jpg)

在谷歌网站上面选择相关字体，引入到项目中即可。

原理：
谷歌字体把中文做了分区处理，根据使用到的字符，加载相应部分，使用的时候一般 20kb 加载多条。

![字体分区](/images/16181294659403.jpg)


## 参考资料
* [衬线体 - 维基百科，自由的百科全书](https://zh.wikipedia.org/wiki/%E8%A1%AC%E7%BA%BF%E4%BD%93)
* [前端css字体调研 · Issue #14 · o2team/H5Skills](https://github.com/o2team/H5Skills/issues/14)
* [aui/font-spider: Smart webfont compression and format conversion tool](https://github.com/aui/font-spider)
* [Browse Fonts - Google Fonts](https://fonts.google.com/)

