---
title: ESLint 自定义规则开发
cateories: 前端
img: ../../coverImages/ESLint.png
categories:
  - front-end
date: 2023-02-16 06:00:00
---

ESLint 是 js 代码格式化工具，能够自动发现并尝试修复代码中的问题，是团队开发的必备工具。大部分时候，使用既定规则即可，业界也有比较多的成熟配置：

- eslint:recommended ESLint内置的推荐规则在么有讲到 所有打钩的就是内置规则
- eslint:all：ESLint 内置的所有规则；
- eslint-config-standard：standard 的 JS 规范；
- eslint-config-prettier：关闭和 ESLint 中以及其他扩展中有冲突的规则；
- eslint-config-airbnb-base：airbab 的 JS 规范；
- eslint-config-alloy：腾讯 AlloyTeam 前端团队出品，可以很好的针对你项目的技术栈进行配置选择，比如可以选 React、Vue（现已支持 Vue 3.0）、TypeScript 等；

本篇并不想讨论这些规则的配置和使用，搜索引擎学习这些内容会更加容易。

## 问题

```
var date = new Date("2023-02-16 06:00");
```
上面的代码在pc上没有大问题，但是在移动端 ios 中会报错，ios 不支持 `2023-02-16 06:00` 这样的写法，需要写成 `2023/02/16 06:00` 这样的形式。

在项目管理中，团队人员多了，很难从口头成面达成一致，最好还是从 ESLint 规则上面去想办法。

## 源码
![](/images/2023/02/16/16765026485289.jpg)

ESLint 第一个 commit 是 2013 年，2013.7.1 发布了 0.0.2 版本，后续虽然一直在更新各种功能，但是核心逻辑其实没有变化，我们看 0.0.2 版本的源码会更加轻松一点。

![](/images/2023/02/16/16765028134191.jpg)

我们熟悉的 `no-console` 等规则在第一版已经实现。我们看一下它的具体实现

```js
module.exports = function(context) {
    return {
        "MemberExpression": function(node) {
            if (node.object.name === "console") {
                context.report(node, "Unexpected console statement.");
            }
        }
    };
};
```

代码很简单，判断节点是否是 `console`,如果是，那么调用 context.report 函数去上报异常。`MemberExpression` 这个方法名称有点奇怪，可能是比较核心的内容。

我们继续梳理代码，很快就会发现这一块内容
![](/images/2023/02/16/16765032551049.jpg)

逻辑非常清晰，读取配置，读取规则，对文件应用规则。
继续跟踪读取文件：

![](/images/2023/02/16/16765033790035.jpg)

jscheck 是一个类，上面挂载了一些方法，verify 似乎是干活的主要部分：
![](/images/2023/02/16/16765034874120.jpg)

部分配置项被我收起，不影响我们核心理解，这里出现了两个 npm 包：
- [esprima](https://github.com/jquery/esprima)
- [astw](https://www.npmjs.com/package/astw)

### esprima
```js
var esprima = require('esprima');
var program = 'const answer = 42';
var program2 = 'console.log(123)';

console.log(JSON.stringify(esprima.parse(program)))
console.log(JSON.stringify(esprima.parse(program2)))
```
输出：
```json
{
  "type": "Program",
  "body": [
    {
      "type": "VariableDeclaration",
      "declarations": [
        {
          "type": "VariableDeclarator",
          "id": {
            "type": "Identifier",
            "name": "answer"
          },
          "init": {
            "type": "Literal",
            "value": 42,
            "raw": "42"
          }
        }
      ],
      "kind": "const"
    }
  ],
  "sourceType": "script"
}
{
  "type": "Program",
  "body": [
    {
      "type": "ExpressionStatement",
      "expression": {
        "type": "CallExpression",
        "callee": {
          "type": "MemberExpression",
          "computed": false,
          "object": {
            "type": "Identifier",
            "name": "console"
          },
          "property": {
            "type": "Identifier",
            "name": "log"
          }
        },
        "arguments": [
          {
            "type": "Literal",
            "value": 123,
            "raw": "123"
          }
        ]
      }
    }
  ],
  "sourceType": "script"
}

```

我们前面遇到的 `MemberExpression` 在这里再次出现，成员表达式节点，即表示引用对象成员的语句。这里是把 js 转换成抽象语法树，后续根据语法树去判读是否和规则匹配。

常见的 AST 节点：
- Identifier
- ImportSpecifier
- ImportDefaultSpecifier
- CallExpression
- MemberExpression
- AssignmentExpression
- ArrayExpression
- VariableDeclaration
- LogicalExpression
- ConditionalExpression
- IfStatement
- Literal
- BinaryExpression
- BinaryOperator
- ExpressionStatement
- ThisExpression
- ObjectExpression
- NewExpression
- Property
- TemplateLiteral
- TemplateElement
- ...

常见的节点比较多，不需要死记硬背，有个大致印象就好。需要生成 ast ，可以使用下面这个网站：
https://astexplorer.net/

可以快速生成查阅 ast 节点。


### astw
esprima 是生成 ast，那么 astw 猜测这里就是遍历 ast ？
```js
var ast = esprima.parse(text, { loc: true, range: true }),
            walk = astw(ast);

        walk(function(node) {
            api.emit(node.type, node);
        });
```

第一次调用，也是做的一些配置合并的操作，返回了 walk 函数。
walk 函数再次调用，会遍历 ast 节点，对于最终子节点，会执行回调函数。
回调函数根据事件 `emit、on` 进一步执行是否需要 `report` 的逻辑。
![](/images/2023/02/16/16765048037892.jpg)

![](/images/2023/02/16/16765047143617.jpg)

到这里，源码分析基本完成。逻辑非常清晰，读取配置，读取自定义规则，配置合并，文件转换为 ast，遍历 ast 节点并和规则做对比。

### 后续 ESLint 迭代
后续 ESLint 越来越复杂，部分开源的内容，已经跟不上 ESLint 的发展，有了一些自己的视线
- [eslint/espree: An Esprima-compatible JavaScript parser](https://github.com/eslint/espree)：Espree 是 esprima 的新的实现，满足了新的 js 语法需求

plugin 机制，你可以直接替换编译器，自己构建语法解析器，ESLint 完全变成工具人，只负责主线流程，其他都可以自己实现，使得社区快速完成了对 `vue、jsx` 等的语法解析。直接干死了 `jslint`。后续 ts 推出了 `TSLint`，但是没多久也放弃了，直接被 `ESLint` 吞并。


## 跑题了？
咦，我们是不是跑题了？看到这里，我还不会写 ESLint 规则嘛。下面其实很简单了。

全局安装脚手架

```js
npm i -g yo
npm i -g generator-eslint
```

生成项目骨架
```js
yo eslint:plugin
```

根据这个骨架，你就可以开发自己的 ESLint 规则，比如 `new Date` 兼容性问题核心代码如下：

```js
NewExpression(node){
    if(node.callee.name === 'Date'){
        if (node.arguments.length && node.arguments[0].value && node.arguments[0].value.includes("-")){
            context.report({
                node,
                message: "new Date('2022-01-01') 横线写法在ios中会报错",
            });
        }
    }
}
```

更多实现细节请参考我对支付宝小程序写的规则
[eslint-plugin-alipay-mini](https://github.com/Yaob1990/eslint-plugin-alipay-mini)：https://github.com/Yaob1990/eslint-plugin-alipay-mini


# 参考资料
[前端 - 13 个示例快速入门 JS 抽象语法树 - 不挑食的程序员 - SegmentFault 思否](https://segmentfault.com/a/1190000015660623)