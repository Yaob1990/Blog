---
title: ts 方法装饰器
categories:
  - front-end
img: ../../coverImages/typescriptLogo.jpg
tags:
  - typescript
date: 2021-07-12 23:30:00
---

装饰器在java中叫注解，体现了面向切面（AOP）的编程思想。

## 方法装饰器
```typescript
// 定义装饰器
export  function logger(target: any, propertyKey: string, descriptor: PropertyDescriptor) {
    console.log("初始化");
    const original = descriptor.value;
    descriptor.value = function (...args: any) {
        console.log(args)
        const result = original.call(this, ...args);
        return result;
    }
}

export default class Index extends Vue {
  private title = "学习装饰器"
  // 使用装饰器
  @logger
  private imgClick(e) {
    console.log("图片被点击")
  }
}
```
### 参数详解：
- target：类，也就是这里的 `Index`
- propertyKey：类属性函数的名称，这里是 `imgClick`
- descriptor：属性描述符，通过descriptor上面的属性，即可实现属性只读，数据重写等功能

使用装饰器后，初始化时候会打印 `初始化`，descriptor.value 获取绑定的函数`imgClick`，对`imgClick`进行重写，打印参数，输出结果。

看起来比较复杂的装饰器，在一步一步的分支之后，也挺好理解的，本质上我们使用方法装饰器，往往是对方法的重写，或者是执行 f(g(fn()))类似的函数。

### 使用场景
1. 防抖节流
2. 埋点
3. ...
任何和主业务无关，可以被提炼出公共函数的场景都可以尝试使用装饰器去实现。