---
title: 五福投放技术实现
cateories: 前端
img: ../../coverImages/wufu.png
categories:
  - front-end
date: 2023-02-26 17:00:00
---


今年支付宝五福在众多小程序中都有接入，技术上接入是无感知的，本篇博客尝试去分析实现这种无感知的接入。

## 分析
小程序约等于 h5，如果我们要实现类似五福的弹框，那么必然要引入一段js，由这段 js 去实现页面的效果。
弹框怎么样保证在最前面，不被页面的元素层级覆盖，我们需要获取页面最大的 `z-index` 值。

## 层级

```js
function getMaxZIndex(){
    return [...document.body.querySelectorAll('*')].reduce((r, e) => Math.max(r, +window.getComputedStyle(e).zIndex || 0), 0)
}
```
`document.body.querySelectorAll('*')` 获取页面所有元素，返回 NodeList 类数组，转化为一般数组之后，通过 `window.getComputedStyle(e).zIndex` 获取最大的层级，在 reduce 函数中对比下大小，返回最大的 `z-index` 层级

通过这部分，我们只需要在最大的 `z-index` 上面加1即可以保证我们的层级在最上面。



## 如何引入 js
弹框已经不成问题，那我们怎么在众多的页面中去引入这段 js 呢？还不能在单体应用维度去操作。
nginx或者网关。我们的 html 文件都会从 nginx 去分发，我们可以从这个口子直接去做文件内容的替换。

nginx 内置的 ngx_http_sub_module 模块可以帮助我们实现这部分功能。

### ngx_http_sub_module：

1）介绍：

ngx_http_sub_module模块是一个过滤器，它修改网站响应内容中的字符串。这个模块已经内置在nginx中，但是默认未安装，需要安装需要加上配置参数：--with-http_sub_module 如果已经安装nginx,只需要再添加这个模块就可以了。

2）常用指令：

2.1 sub_filter指令： sub_filter string（原字符串） replacement（用于替换的字符串）;
用于设置需要使用说明字符串替换说明字符串.string是要被替换的字符串，replacement是 新的字符串，它里面可以带变量。

2.2 sub_filter_last_modified指令： sub_filter_last_modified on | off;
用于设置网页内替换后是否修改 可在nginx.conf的 http, server, location三个位置配置使 用，默认值是off；

2.3 sub_filter_once指令：sub_filter_once on | off;
用于设置字符串替换次数，默认只替换一次。如果是on，默认只替换第一次匹配到的到字 符，如果是off，那么所有匹配到的字符都会被替换；

2.4 sub_filter_types指令：sub_filter_types * 
用于指定需要被替换的MIME类型,默认为“text/html”，如果制定为*，那么所有的；

3）说明：
3.1以上指令可在nginx.conf的http, server, location三个位置配置使用；
3.2此模块替换不区分大小写；
3.3支持中文替换；

```bash
location ^~/im/ {
    proxy_pass http://p.qiao.baidu.com;
    add_header Access-Control-Allow-Origin *;
    sub_filter '</body>' '<script src="example.com" /></body>';##将响应内容中</body>替换为对应的 script 标签
    sub_filter_once off;####所有匹配到的都替换
}
```

## 总结
实现并不算太复杂，很像 http 时代的互联网运营商插入的小广告，也像 chrome 的插件机制。现在的实现还比较粗糙，在 nginx 层面，如果有网关，应该会更容易一点。
技术是一把双刃剑，想用好这把剑，要不断地修炼自己。


## 参考资料
- [nginx替换响应内容 - kenwar - 博客园](https://www.cnblogs.com/kenwar/p/8296508.html)