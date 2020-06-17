---
title: 丰云行app 探索
date: 2020-06-17 08:21:37
categories: 前端
img: ../../coverImages/fyx.jpg
tags:
  - axios
  - promise
typora-root-url: ..\..
---

广汽丰田的风云行app，提供的车联网功能还是挺好用的，但是只能提供车辆的位置信息，不能提供行驶轨迹等功能。于是抓包看了下他的功能逻辑。

### 登录

```js
// 登录地址（post）：
https://carapp.gtmc.com.cn/api/appgtmc/reg/action/AppUserInfoAction/appUserLogin.json
// 参数（body）
accessToken=&appVersion=4.3.0&checkCode=&deviceId=&deviceType=2&distinctId=&loginType=2&password=&phoneName=&phoneNumber=&pushId=
```

返回值：

```
{
    "data": {
        "jwt": "",
        "rData": {
            "birthday": "",
            "passWord": "111",
            "communtiyFlag": "0",
            "dealerName": "",
            "address": "",
            "kickOut": true,
            "dealerCode": "",
            "sex": "null",
            "resultCode": "",
            "description": "",
            "userId": 111,
            "uuid": "111",
            "telPhone": "111",
            "name": "1",
            "status": "0",
            "username": "11"
        }
    },
    "success": true,
    "resultCode": 200,
    "elapsedMilliseconds": 0
}
```

jwt 就是登录凭证，有了这个登录凭证后续的请求都可以进行鉴权。

这里后端返回了`passWord`字段，个人觉得非常不合适。后端要么是直接返回的用户数据，要么就没有做非对称加密，直接数据库解码出用户的密码字段！通过网络劫持，很容易就嗅探出用户的密码了。

给广汽的邮箱和微信反应过这个问题，没有得到任何回应....



### 控制密码校验

```
// get 请求
https://carapp.gtmc.com.cn/api/vhcApp/controlpwd/checkControlPwd?phone=&pwd=
// header 
Authorization:{{jwt}}
```

通过用户的手机号码和控制密码,jwt校验,进行用户身份识别.

返回:

```js
{
    "resultCode": 200,
    "errMsg": null,
    "elapsedMilliseconds": 35,
    "data": {
        "code": "1"
    },
    "success": true
}
```

`code:1` 校验通过,经过测试,这个校验状态的有效期约为三分钟



### 获取车辆位置

```js
// get 请求(vin 是车架号码)
https://carapp.gtmc.com.cn/api/vhcApp/lookForCar/fixedPosition?vin=
// header 
Authorization:{{jwt}}
```



返回结构:

```js
{
    "resultCode": 200,
    "errMsg": null,
    "elapsedMilliseconds": 1009,
    "data": {
        "remissId": null,
        "latitude": "100",
        "being": 0,
        "longitude": "200",
        "status": 1
    },
    "success": true
}
```

其中 latitude longitude 就是车辆的经纬度了.



### 位置上报

以上的信息似乎就可以获取车辆的位置信息,画出行驶轨迹了。

然后，我在车辆行驶中时候抓包时候，发现车辆的位置一直是上一次停车的那个点。推测，车辆位置并不是实时上报，而是车辆停止，熄火后上报一次，功能也只用来追踪车辆的最终停止位置。



### 还能做什么

抓包研究的过程还是非常有意思的。虽然我们最初的结果没有达到，但是我们用这些数据依然可以做很多有意思的事情，比如用`flutter`把这些功能重写一遍？