---
title: 运动步数修改器 web 版本
date: 2021-05-22 23:30:00
categories: 前端
img: ../../coverImages/step.jpeg
---

![](/images/16216988358049.jpg)


## 思路
1. 修改本地步数，比如健康的数据，比较麻烦，没有找到特别方便的方法。
2. 摇步器，慢慢慢，工作时间内只能摇 20 公里左右
3. 通过第三方同步数据，比如乐心健康，小米运动等通过接口同步到微星、支付宝等平台

## 实现
实现其实挺简单的，github 上面的代码也挺多，包括 nodejs 的实现，shell 的实现，python 的实现。
这里我们选择了 nodejs 的实现，前端使用 react + antd，后端使用 nestjs，使用时候，直接返回日志给用户，方便用户查看失败详情。
代码写的比较烂，就暂时不开源了。
访问地址：https://walk.jscoder.com/

## 使用说明
1. 注册小米运动 app
2. 小米运动 app，我的-第三方接入，设置接入的第三方平台，支持微信、QQ、支付宝、阿里体育
3. https://walk.jscoder.com/ 输入小米运动的账号密码，需要的步数，点击确定

### 注意：
1. 不要频繁修改你的数据
2. 不要在 0:00 ~ 5:00 这个时间段修改你的数据
3. 我不会记录你的密码，但是依然建议你使用临时密码


## 参考
 
- [Devifish/sport-editor: 通过小米运动API实现的自动刷运动步数工具😒（可同步到微信、支付宝）](https://github.com/Devifish/sport-editor)
- [mixool/mimotion: 小米运动 微信步数 支付宝步数](https://github.com/mixool/mimotion)
- [lisztomania-Zero/SportsChange: 微信、支付宝、QQ、阿里体育运动步数修改](https://github.com/lisztomania-Zero/SportsChange)
