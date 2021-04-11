---
title: 前端怎么样处理emoji表情
date: 2020-01-18 20:40:00
categories: 前端
img: ../../coverImages/emoji.jpg
tags:
  - JSON
  - emoji
typora-root-url: ..\..\..
---

> 一个 bug 的心路历程...
>
> 😀：新项目耶 ✌
>
> 🤔：怎么做是最佳实践呢
>
> 😁：终于好了
>
> 😎：测试通过，顺利上线
>
> 😱🥵😡：什么，线上 bug！

一般而言，不推荐前端处理 emoji，最好的方式还是把数据库修改成 `utf8mb4`格式，但是，如果数据库不在你这里呢？如果是一个延续性项目呢...那前端可能没办法,就要自己去处理 emoji 了。

### emoji 是什么

emoji 是字符，不是图片。emoji 使用 Unicode 编码方式，可以方便的转换成其他格式。

[维基百科关于 emoji 的介绍](https://zh.wikipedia.org/wiki/繪文字)

[charbase.com](https://charbase.com/1f602-unicode-face-with-tears-of-joy) ，网站提供了，各种格式 emoji 编码的查看方式：

![笑哭](/images/image-20200118191909665.png)

上图可以方便的查看笑哭的 Unicode 编码：U+1F602，Javascript Escape："\ud83d\ude02"等。

### 常见 emoji 处理方式

1. 后台更改数据库格式为`utf8mb4`格式
2. 使用文字替换，比如微信，QQ 等是使用`[笑哭]`格式表示 😂，
3. 文字替换，但是不使用系统的 emoji，而是根据文字显示图片，实现多端显示统一
4. 使用系统自带的 emoji 表情

### emoji 常用转换函数

转换函数，涉及到 unicode 的编码算法，属于专有领域知识，比较复杂，需要的时候复制就行了...不必深究。

方法一：转成Unicode

```js
// https://github.com/channg/umoji/blob/master/src/emojiToUnicode.js
// 转换为js编码方式  😀=>"\ud83d\ude00"
function emojiToUnicode(emoji) {
  let backStr = "";
  if (emoji && emoji.length > 0) {
    for (const char of emoji) {
      const index = char.codePointAt(0);
      if (index > 65535) {
        const h =
          "\\u" + (Math.floor((index - 0x10000) / 0x400) + 0xd800).toString(16);
        const c = "\\u" + (((index - 0x10000) % 0x400) + 0xdc00).toString(16);
        backStr = backStr + h + c;
      } else {
        backStr = backStr + char;
      }
    }
    console.log(backStr);
  }
  return backStr;
}


```

方法二：转成HTML实体编码

**不推荐！！！**

如果显示HTML编码，那么在`vue`框架中必须使用`v-html`,这是非常危险的，很容易导致`xss`攻击！

```javascript
// https://jordonwang.github.io/2018/06/06/emoji-string/
// 转换为HTML实体字符
function emojiToHTMLEscape(str) {
  var patt = /[\ud800-\udbff][\udc00-\udfff]/g; // 检测utf16字符正则
  str = str.replace(patt, function(char) {
    var H, L, code;
    if (char.length === 2) {
      H = char.charCodeAt(0); // 取出高位
      L = char.charCodeAt(1); // 取出低位
      code = (H - 0xd800) * 0x400 + 0x10000 + L - 0xdc00; // 转换算法
      return "&#" + code + ";";
    } else {
      return char;
    }
  });
  return str;
}
```



如果想深入研究，可以参考：

[移动前端手机输入法自带 emoji 表情字符处理](https://blog.csdn.net/binjly/article/details/47321043)

[Unicode 与 JavaScript 详解](http://www.ruanyifeng.com/blog/2014/12/unicode.html)

### bug 场景及解决办法

```javascript
// 数据来自服务端,我传递时候做JSON编码，\被转义为\\
let serverValue = "\\ud83d\\ude00\\ud83d\\ude00";
let value = JSON.parse(serverValue);
<div>{value}</div>;
```

看起来是一切正常，没有问题。我传递给后端的数据，做`JSON.Stringfy(value)`编码，拿到时候，再对字符串做 `JSON.parse(value)`解码。

但是，线上报错。仔细研究发现，有些数据，并不是我传递给后端的，而这些数据里面含有 JSON 的非法字符，导致 JSON 报错。

解决方式也简单：

1. 不使用 JSON.parse 解码，直接替换\\\为\\,就可以直接显示在页面上
2. 如果使用 JSON.parse 解码，加个 try catch



似乎问题解决了，但是JS似乎并不能很好的处理\\\为\\，[stackoverflow](https://stackoverflow.com/questions/33685680/emoji-surrogate-string-with-javascript-how-to-parse)这个方法正则方法可以：

```javascript
function parseUnicode(str){
    var r = /\\u([\d\w]{4})/gi;
    str = str.replace(r, function (match, grp) {
        return String.fromCharCode(parseInt(grp, 16)); } );
    return str;
}
```



在线测试：

{% jsfiddle wukong/6e1fjz4w/23 js,html,result dark 100% 500 %}



### JSON 对字符串的处理

前端使用 json，基本都是对 object 进行编码，解码，很少对字符串进行编码解码。如果 JSON.parse()遇到'\'，往往会报错，基本上都需要转义为‘’\\\‘

自己对`JSON`规范并不熟悉，关于`JSON`以后单独再写一篇。

![JSON 转义测试](/images/image-20200118203439540.png)

<img src="/images/json.png" alt="轨道图" style="zoom: 33%;" />

### 总结

问题最终定位在 JSON 的转义上面，虽然天天在用 JSON，但是并不理解，查了很多资料，还是一星半解，这篇博客是很难写好了...
