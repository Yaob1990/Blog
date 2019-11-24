---
title: vagrant 学习记录
date: 2019-01-06 18:27:43
categories: 工具
tags:
  - vagrant
---

Vagrant 是virtualBox的命令行管理工具。
以前在mac平台我会用mamp pro 作为php的开发环境，但是非正版毕竟不太好，而且开发环境不贴近实际线上，折腾了一下，使用 vagrant 成功代替mamp pro。

### 1. 安装vagrant

    `brew cask install virtualbox` 安装 virtualbox
    `brew cask install vagrant` 安装 vagrant

### 2. 使用镜像初始化虚拟机

    下载地址找个百度盘吧，官方的我实在是没有拖动。。。一点都没动。。。
    `vagrant init precise64` 其中 `precise64` 表示你的镜像文件
    
###  3. 常用设置
    
```
vagrant up  启动虚拟机
vagrant ssh ssh进入虚拟机
```

配置文件：
需要设置的地方其实很少，也就是网络端口转发和本地其他机器访问的公共ip
`config.vm.synced_folder`表示本地和虚拟机同步的文件夹。
```
  config.vm.network "forwarded_port", guest: 80, host: 80
  config.vm.network "forwarded_port", guest: 8888, host: 8888
  config.vm.network "forwarded_port", guest: 888, host: 888
  config.vm.network "public_network", ip: "192.168.1.120"
  config.vm.synced_folder "../data", "/vagrant_data"
```

在开发中，发现php不能自动生成tpl模板文件，需要设置同步文件夹的权限

```
config.vm.synced_folder   
   "your_folder"(必须)   //物理机目录，可以是绝对地址或相对地址，相对地址是指相对与vagrant配置文件所在目录
  ,"vm_folder(必须)"    // 挂载到虚拟机上的目录地址
  ,create(boolean)--可选     //默认为false，若配置为true，挂载到虚拟机上的目录若不存在则自动创建
  ,disabled(boolean):--可选   //默认为false，若为true,则禁用该项挂载
  ,owner(string):'www'--可选   //虚拟机系统下文件所有者(确保系统下有该用户，否则会报错)，默认为vagrant
  ,group(string):'www'--可选   //虚拟机系统下文件所有组( (确保系统下有该用户组，否则会报错)，默认为vagrant
  ,mount_options(array):["dmode=775","fmode=664"]--可选  dmode配置目录权限，fmode配置文件权限  //默认权限777
  ,type(string):--可选     //指定文件共享方式，例如：'nfs'，vagrant默认根据系统环境选择最佳的文件共享方式
```
我的配置：
`config.vm.synced_folder "../data", "/vagrant_data",create: true, owner:"www", group: "www"`

### 4. 安装宝塔面板
对我我等弱鸡鸡，宝塔面板还是很方便的。直接安装官方说明安装即可。注意，最后安装成功的ip是外网的ip，我们需要本地地址去访问。

### 5. 配置网站
宝塔已经可以配置网站了。但是设置了之后，并不能访问。
需要去hosts 里面配置下网络地址，推荐使用[SwitchHosts](https://oldj.github.io/SwitchHosts/)去管理hosts。
eg：

```
# SwitchHosts!

# vagrant
127.0.0.1 www.test.com
127.0.0.1 www.btadmin.com
```

这时候就可以用`www.test.com`访问测试网站，用`www.btadmin.com`访问宝塔管理页面（需要在宝塔中配置）。
