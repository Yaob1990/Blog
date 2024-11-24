---
title: HomeAssistant 智能家居入门
categories:
  - anything
img: ../../coverImages/smarthome.jpg
tags:
  - 智能家居
date: 2022-12-26 11:34
---


智能家居有很多厂商的方案，但是基本都是打包方案，很难在 AB 厂家之间做联动，并且这些方案都依赖厂家的服务器，服务器出现问题，智能变智障。

1. 依赖厂商服务器
2. 无隐私可言，数据被厂商分析
3. 难以多品牌联动
4. 难以定制化

## Home Assistant
Home Assistant 诞生于 2013 年，是一款基于 Python 开发的 智能家居 开源系统，他的主要功能就是可以将不同品牌的智能家居设备连接起来互联互通。
这个开源项目已经发展了已经快十年了，同时期的其他类似项目都已经雨打风吹去了，目前看 Home Assistant 更新还是很频繁的，每个月都有好几个版本发布。

![](/images/2022/12/26/16720467288022.jpg)


目前我手上绝大部分智能家居设备都可以通过 HomeAssistant 接入。
1. 群晖：监听温度，硬盘状态等
2. 美的燃气热水器：水温，能耗
3. 小米、涂鸦开关：开关通断、能耗
4. 米家空调伴侣：自动化空调设置
5. 小米手机：判断是否在家，自动化灯具
6. Apple tv：自动化开启关闭等
7. 涂鸦窗帘机：自动化窗帘开启关闭

![](/images/2022/12/26/16720467288022.jpg)


### 安装
1. Docker 安装，注意这个版本是半残废版本，没有 addon-on,
2. 虚拟机安装，如果使用 unraid，群晖，建议安装这个版本，功能最全
3. 购买成品，一般是魔百盒刷机之后，刷入了 Debian，然后安装的 HomeAssistant
![](/images/2022/12/26/16720467533799.jpg)

![](/images/2022/12/26/16720467618102.jpg)



## zigBee
ZigBee 实际是一种短距离、低功耗的无线通信技术，名称来源于 ZigZag——一种蜜蜂的肢体语言。当蜜蜂新发现一片花丛后会用特殊舞蹈来告知同伴发现的食物种类及位置等信息，是蜜蜂群体间一种简单、高效的传递信息方式，因此 ZigBee 也被称为“紫蜂协议”。

技术优势：
1. 超低功耗
2. 安全性高
3. 自组网，可容纳电子设备多
4. 可靠性高

不推荐大量设备通过 wifi 接入，使用 zigBee 稳定性要比 wifi 高很多，目前我的新购入设备都是 zigBee。

## Zigbee2mqtt

Zigbee2mqtt 是一个将 Zigbee 协议转化成 MQTT 的桥接工具, 从而接入智能家居平台。推荐直接购买集成了 zigBee 模块的盒子。
Zigbee2mqtt 支持的设备多大 2572 种，并且还在持续增加中。

![](/images/2022/12/26/16720467728426.jpg)

![](/images/2022/12/26/16720467818118.jpg)

![](/images/2022/12/26/16720467884269.jpg)


## 米家设备——Xiaomi Miot Auto

[MIoT-Spec](https://iot.mi.com/new/doc/design/spec/overall) 是小米 IoT 平台根据硬件产品的联网方式、产品功能的特点、用户使用场景的特征和用户对硬件产品使用体验的要求，设计的描述硬件产品功能定义的标准规范。

本插件利用了 **miot** 协议的规范，可将小米设备自动接入 [HomeAssistant](https://www.home-assistant.io/)，目前已支持大部分小米米家智能设备。且该插件支持 HA 后台界面集成，无需配置 yaml 即可轻松将小米设备接入 HA。

## [支持的设备](https://github.com/al-one/hass-xiaomi-miot/issues/12)

- 🔌 [插座](https://home.miot-spec.com/s/plug) / [开关](https://home.miot-spec.com/s/switch)
- 💡 [智能灯](https://home.miot-spec.com/s/light)
- ❄️ [空调](https://home.miot-spec.com/s/aircondition) / [空调伴侣](https://home.miot-spec.com/s/acpartner) / [红外空调](https://home.miot-spec.com/s/miir.aircondition)
- 🌀 [风扇](https://home.miot-spec.com/s/fan) / [凉霸](https://home.miot-spec.com/s/ven_fan)
- 🛀 [浴霸](https://home.miot-spec.com/s/bhf_light) / 🔥 [取暖器](https://home.miot-spec.com/s/heater) / [温控器](https://home.miot-spec.com/s/airrtc)
- 📷 [摄像头](https://home.miot-spec.com/s/camera) / [猫眼/可视门铃](https://home.miot-spec.com/s/cateye) [❓️](https://github.com/al-one/hass-xiaomi-miot/issues/100#issuecomment-903078604)
- 📺 [电视](https://home.miot-spec.com/s/tv) / 📽️ [投影仪](https://home.miot-spec.com/s/projector) / [机顶盒](https://home.miot-spec.com/s/tvbox)
- 🗣️ [小爱音箱](https://home.miot-spec.com/s/wifispeaker) [❓️](https://github.com/al-one/hass-xiaomi-miot/issues/100#issuecomment-885989099)
- 🎮️ [万能遥控器](https://home.miot-spec.com/s/chuangmi.remote) [❓️](https://github.com/al-one/hass-xiaomi-miot/commit/fbcc8063783e53b9480574536a034d338634f4e8#commitcomment-56563663)
- 🔐 [智能门锁](https://home.miot-spec.com/s/lock) / 🚪 [智慧门](https://home.miot-spec.com/s/door)
- 👕 [洗衣机](https://home.miot-spec.com/s/washer) / [干衣机](https://home.miot-spec.com/s/dry) / [冰箱](https://home.miot-spec.com/s/fridge)
- 🚰 [净水器](https://home.miot-spec.com/s/waterpuri) / [饮水机](https://home.miot-spec.com/s/kettle)
- ♻️ [空气净化器](https://home.miot-spec.com/s/airpurifier) / [新风机](https://home.miot-spec.com/s/airfresh) / [油烟机](https://home.miot-spec.com/s/hood)
- 🌡 [温湿度传感器](https://home.miot-spec.com/s/sensor_ht) / [水侵传感器](https://home.miot-spec.com/s/flood) / [烟雾传感器](https://home.miot-spec.com/s/sensor_smoke)
- 🥘 [电饭煲](https://home.miot-spec.com/s/cooker) / [压力锅](https://home.miot-spec.com/s/pre_cooker)
- 🍲 [电磁炉](https://home.miot-spec.com/s/ihcooker) / [烤箱](https://home.miot-spec.com/s/oven) / [微波炉](https://home.miot-spec.com/s/microwave)
- 🍗 [空气炸锅](https://home.miot-spec.com/s/fryer) / [多功能锅](https://home.miot-spec.com/s/mfcp)
- 🍵 [养生壶](https://home.miot-spec.com/s/health_pot) / ☕️ [咖啡机](https://home.miot-spec.com/s/coffee)
- 🍹 [破壁机](https://home.miot-spec.com/s/juicer) / [搅拌机](https://home.miot-spec.com/s/juicer) / [果蔬清洗机](https://home.miot-spec.com/s/f_washer)
- ♨️ [热水器](https://home.miot-spec.com/s/waterheater) / [洗碗机](https://home.miot-spec.com/s/dishwasher) / [足浴器](https://home.miot-spec.com/s/foot_bath)
- 🦠 [消毒柜](https://home.miot-spec.com/s/steriliser)
- 🪟 [窗帘电机](https://home.miot-spec.com/s/curtain) / [开窗器](https://home.miot-spec.com/s/wopener) / [晾衣机](https://home.miot-spec.com/s/airer)
- 🧹 [扫地/扫拖机器人](https://home.miot-spec.com/s/vacuum) / [擦地机](https://home.miot-spec.com/s/.mop)
- 💦 [加湿器](https://home.miot-spec.com/s/humidifier) / [除湿器](https://home.miot-spec.com/s/derh) / [除味器](https://home.miot-spec.com/s/diffuser)
- 🍃 [空气检测仪](https://home.miot-spec.com/s/airmonitor) / 🪴 [植物监测仪](https://home.miot-spec.com/s/plantmonitor)
- 🛏 [电动床](https://home.miot-spec.com/s/bed) / [电热毯/水暖床垫](https://home.miot-spec.com/s/blanket) / 😴 [睡眠监测仪](https://home.miot-spec.com/s/lunar)
- 💆 [按摩椅](https://home.miot-spec.com/s/massage) / [按摩仪](https://home.miot-spec.com/s/magic_touch)
- 🏃 [走步机](https://home.miot-spec.com/s/walkingpad) / [跑步机](https://home.miot-spec.com/s/treadmill) / [升降桌](https://home.miot-spec.com/s/desk)
- 🚽 [马桶(盖)](https://home.miot-spec.com/s/toilet) /️ [毛巾架](https://home.miot-spec.com/s/.tow) /️ 🪥 [牙刷](https://home.miot-spec.com/s/toothbrush)
- 🐱 [宠物喂食器](https://home.miot-spec.com/s/feeder) / ⛲ [宠物饮水机](https://home.miot-spec.com/s/pet_waterer) / 🐟 [鱼缸](https://home.miot-spec.com/s/fishbowl)
- 🦟 [驱蚊器](https://home.miot-spec.com/s/mosq) / [消毒/灭菌灯](https://home.miot-spec.com/s/s_lamp)
- 🚘 [智能后视镜](https://home.miot-spec.com/s/rv_mirror) / [抬头显示HUD](https://home.miot-spec.com/s/hud)
- ⌚️ [智能/儿童手表](https://home.miot-spec.com/s/watch) / [手环](https://home.miot-spec.com/s/bracelet)
- 🚶 [人体传感器](https://home.miot-spec.com/s/motion) / 🧲 [门窗传感器](https://home.miot-spec.com/s/magnet) [❓️](https://github.com/al-one/hass-xiaomi-miot/issues/100#issuecomment-909031222)
- 📳 [动静贴](https://home.miot-spec.com/s/vibration)
- 🌐 [路由器](https://home.miot-spec.com/s/router) / 🖨 [打印机](https://home.miot-spec.com/s/printer)

接入之后，几乎不需要任何配置就可以直接使用，并且和手机使用米家不冲突。

## ESP32

上面这些东西，包括进一步的自动化，可以玩一个月了，如果还不过瘾，可以继续看看 ESP32，自己动手做智能家居。

ESP32 说的是主板上的主控芯片，是由我国的乐鑫公司 (ESPRESSIF) 继 ESP8266 芯片后推出的又一款集成 WiFi 功能的微控制器。ESP32 芯片或模组具有下列特点：

处理器：Tensilica LX6 双核处理器（一核处理高速连接；一核独立应用开发）
主频：32 位双核处理器，CPU 正常工作速度为 80 MHz，最高可达 240 MHz
SRAM：520KB，最大支持 8 MB 片外 SPI SRAM
Flash：最大支持 16 MB 片外 SPI Flash
WiFi 协议：支持 802.11 b/g/n/d/e/i/k/r 等协议，速度高达 150 Mbps
频率范围：2.4~2.5 GHz
蓝牙协议：支持蓝牙 v4.2 完整标准，包含传统蓝牙 (BR/EDR) 和低功耗蓝牙 (BLE)
同时他还具备丰富的外设接口：比如 GPIO、ADC、DAC、SPI、I²C、I²S、UART 等常用接口一个不少


### 独立网关

![](/images/2022/12/26/16720468061875.jpg)

![](/images/2022/12/26/16720468142750.jpg)


小黄鱼有很多出售这种 zigBee 带网关的，就是使用 ESP32 来实现的，ESP32 运行服务器，mqtt 等内容，对外发送报文，这样就可以不走 H omeAssistant 类似平台，但是弱点也很明显，支持的设备有限，每次新增设备类型，就需要刷新固件，或者自己编译固件，如果对方固件不开源，那可玩性就大打折扣。

### 自己做摄像头
[ESP32 CAM通过ESPHome连接到Homeassistant - 知乎](https://zhuanlan.zhihu.com/p/577606023)


## 参考资料

* [ESP32 CAM通过ESPHome连接到Homeassistant - 知乎](https://zhuanlan.zhihu.com/p/577606023)
* [小米IoT开发者平台](https://iot.mi.com/new/doc/introduction/main)
* [Home Assistant](https://www.home-assistant.io/)
* [Home | Zigbee2MQTT](https://www.zigbee2mqtt.io/)
* [小米智能家居第三方控制之一：提取米家API并控制-小伟博客](https://www.xiaoweigod.com/network/2235.html)
