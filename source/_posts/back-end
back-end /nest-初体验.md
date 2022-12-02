---
title: nest 初体验
categories:
  - |-
    back-end
    back-end 
img: ../../coverImages/nest.svg
tags:
  - nest
date: 2020-02-23 17:07:03
---



假期中，学习了`nest`这个node后端框架。感觉整体比较简约，学习相对来说，比较容易。本篇，写一下自己对这个框架的理解。

### 特点

1. 类angular的代码组织形式:`imports、module、service`
2. 基于`typescript`，面向未来，特别是基于装饰器的模式，代码变得非常简洁
3. 底层基于`express`,支持更换为`fastify`平台，坐拥丰富的`express`中间件
4. 拒绝刀耕火种，一直没有太深入的学习`koa`的一个重要原因。koa框架太简单，基本什么都做不了，都需要引入中间件，封装代码。`nest`对常用操作，做了丰富的封装，简单易上手。
5. `node` 中的`spring`，`typescript`+`ioc`,值得学习~



### 入门安装

1. `npm i -g @nestjs/cli`
2. `nest new project-name`



### 文档理解

[官方文档](https://docs.nestjs.cn/6/firststeps) 写的还不错，看一遍，基本也能理解。不准备重复官方文档，针对文档的各个模块写一点自己的理解。

#### 控制器（controller）

可以看成是路由控制器，负责传入的请求和客户端的返回响应。

如果返回对象或者数组格式，会被自动包装成`JSON`格式。

#### 提供者（Providers）

Provider只是一个用 `@Injectable()` 装饰器注释的类。通过Controller 的 `constructor`注入，并实例化。不需要手动实例化，也就是java中常说的依赖注入。

一般用来处理业务逻辑，数据库交互。

#### 中间件

中间件是在路由处理程序 **之前** 调用的函数。本质上，就是路由之前的函数调用。

#### 管道

* **转换**：管道将输入数据转换为所需的数据输出
* **验证**：对输入数据进行验证，如果验证成功继续传递; 验证失败则抛出异常;

感觉`ValidationPipe` 用的会多一点，既可以用来做验证，也可以做转换。

#### 守卫

守卫是一个使用 `@Injectable()` 装饰器的类。 守卫应该实现 `CanActivate` 接口。

一般用来做鉴权，通过守卫函数，返回`True`或者`False`,实现鉴权。



### 总结

`nest`现在每周20k的下载量，是koa的一半，但是他是一个面向未来的框架。基于typescript和类spring的架构，值得我投入时间去学习琢磨。

学习`nest`还有一个原因,假期尝试看了`java`的`spring boot`,概念实在是太繁杂了=-=，奴家实在是不会啊~

好啦，今年的个人项目后台就暂定为`nest`呐~