---
title: 深入浅出 nodejs
categories:
  - reading
img: ../../coverImages/book.jpeg
tags:
  - 阅读
date: 2022-01-23 09:00:00
---

![nodejs](/images/2022/01/23/nodejs.jpeg)

## 没有白看的书
这本书买了五六年了，以前硬着头皮看，做的笔记，批注，到处是困惑，没有白走的路，学不会的总有一天我会把你学会，学不会的，只是必不可少的铺垫。

最近定位 H5-dooring 的server 端问题，才把这本书看完，挺好的一本中文技术书。以前 node 没有经验，完全看不懂，这两年在 nestjs 上面花了比较多的时间，初略写了一点 node 的东西，再回头看这本书，才看懂了一点。

书的章节并不是顺序的，可以跳着看，第二章 C++ 模块现在还是看不懂，索性直接跳过。

## 笔记

### 第五章 内存控制

* v8 有内存限制，32位系统下约为 0.7G，64位系统下约为 1.4G，使用超过限制会导致进程退出。
* v8 的内存分为新生代和老生带。
    * 新生代主要通过 Scavenge 算法，通过牺牲空间换区时间的方式进行垃圾回收。
    * 老生代主要通过 Mark—Sweep（标记清除，只清除死亡对象，较快，导致内存空间不连续），在空间不足以使用 Mark-Sweep 时候，使用Mark-Compact（标记整理，较慢，内存空间连续） 进行垃圾回收。
* Buffer 对象不同于其他对象，不经过 v8 的内存分配机制，不会有堆内存大小的限制

### 第六章 理解 Buffer

* Buffer 是一个类 Array 的对象，但它主要用于操作字节，是二进制数据。
* Buffer 支持的编码类型可以通过 `toString` 方法转为字符串，不支持的编码，如 GBK，GB2312 会乱码
* 字符宽度，导致 Buffer 不能直接破解，要使用 `Buffer.concat` 结合数组操作进行拼接。
* Buffer 传输性能比直接传递字符串要高


### 第七章 网络编程

* 三次握手通过 `socket` 实现，node 中也不例外


### 第九章 玩转进程

* 创建子进程
    * `spawn()`: 启动一个子进程来执行命令
    * `exec()`: 启动一个子进程来执行命令，与 `spawn()` 不同的是其接口不同，它有一个回调接口获悉子进程的情况
    * `execFile()`: 启动一个子进程来执行可执行文件
    * `fork()`: 与 `spawn()` 类似，不同点在于它创建 Node 的子进程只需要指定要执行的 JavaScript 文件模块即可
* 进程间通讯
    * IPC
    * 句柄传递
* 状态共享
    * Redis 等
    * 主动通知
    * Cluster（Pm2 用的它）

## 总结
这本书遇到问题的时候可以多翻翻，各方面都有涉及，当然毕竟是块 10 年的书了，底层，方法论都没有问题，具体的工具性的东西可能需要多搜索下。