---
title: 深入理解 ts 中的 utility 类型
categories:
  - front-end
tags:
  - Typescript
img: ../../coverImages/typescriptLogo.jpg
date: 2021-05-25 07:30:00
---

平时开发中，有些场景总有种蹩手蹩脚的感觉，看到 Utility Types 才知道自己 native 了，很多场景 ts 都帮我们想好了。

## Partial<Type>
设置 Type 某个类型为可选，并返回部分 Type 类型。

### 源码：
```ts
type Partial<T> = { [P in keyof T]?: T[P]; };
```

### 例
```ts
interface Todo {
  title: string;
  description: string;
}

function updateTodo(todo: Todo, fieldsToUpdate: Partial<Todo>) {
  return { ...todo, ...fieldsToUpdate };
}

const todo1 = {
  title: "organize desk",
  description: "clear clutter",
};

const todo2 = updateTodo(todo1, {
  description: "throw out trash",
});
```
## Required<Type>
Type 类型里面的某个类型为必要类型（必选项）。

### 源码

```ts
type Required<T> = {
    [P in keyof T]-?: T[P];
};
```
`-?` 表示移除可选性`?`，这里移除指把可选项改为必选项，而不是移除可选项。

### 例

```ts
interface Props {
  a?: number;
  b?: string;
}

const obj: Props = { a: 5 };
// 报错，确少 b 类型
const obj2: Required<Props> = { a: 5 };
```

## Readonly<Type>
只读属性，不可改写。

### 源码

```ts
type Readonly<T> = { readonly [P in keyof T]: T[P]; };
```
### 例
```ts
interface Todo {
  title: string;
}

const todo: Readonly<Todo> = {
  title: "Delete inactive users",
};
// 报错，不能被重新赋值
todo.title = "Hello";
```

## Record<Keys,Type>
将 `Kyes` 的类型转化为 `Type` 的类型，类似于类型扩展。

### 源码

```ts
type Record<K extends keyof any, T> = {
    [P in K]: T;
};
```

### 例

```ts
interface CatInfo {
  age: number;
  breed: string;
}

type CatName = "miffy" | "boris" | "mordred";

const cats: Record<CatName, CatInfo> = {
  miffy: { age: 10, breed: "Persian" },
  boris: { age: 5, breed: "Maine Coon" },
  mordred: { age: 16, breed: "British Shorthair" },
};
// const cats: Record<CatName, CatInfo>
cats.boris;
```

## Pick<Type, Keys>
将 `Type` 的部分类型 `Keys` 挑出来，返回这部分类型。（拣选属性）

### 源码

```ts
type Pick<T, K extends keyof T> = {
    [P in K]: T[P];
};
```

### 例

```ts
interface Todo {
  title: string;
  description: string;
  completed: boolean;
}


// type TodoPreview = { title: string, completed: boolean }
type TodoPreview = Pick<Todo, "title" | "completed">;

const todo: TodoPreview = {
  title: "Clean room",
  completed: false,
};
// const todo: TodoPreview
todo;
```

## Omit<Type, Keys>
移除 `Type` 类型中的 `Keys` 类型，返回新的类型。和 `Pick` 含义相对。

### 源码

```ts
type Omit<T, K extends keyof any> = Pick<T, Exclude<keyof T, K>>;
```

### 例

```ts
interface Todo {
  title: string;
  description: string;
  completed: boolean;
  createdAt: number;
}

type TodoPreview = Omit<Todo, "description">;

const todo: TodoPreview = {
  title: "Clean room",
  completed: false,
  createdAt: 1615544252770,
};

todo;
 
const todo: TodoPreview

type TodoInfo = Omit<Todo, "completed" | "createdAt">;

const todoInfo: TodoInfo = {
  title: "Pick up kids",
  description: "Kindergarten closes at 5pm",
};
// const todoInfo: TodoInfo
todoInfo;
```
## Exclude<Type, ExcludedUnion>
从 `Type` 中排除可分配给 `ExcludedUnion` 的属性，剩余的属性构成新的类型

### 源码

```ts
type Exclude<T, U> = T extends U ? never : T;
```

### 例

```ts
// T0 = "b" | "c"
type T0 = Exclude<"a" | "b" | "c", "a">;
```
## Extract<Type, Union>
从 `Type` 中抽出可分配给 `Union` 的属性构成新的类型(交集)。与Exclude相反。

### 源码

```ts
type Extract<T, U> = T extends U ? T : never;
```

### 例

```ts
// T0 = "a"
type T0 = Extract<"a" | "b" | "c", "a" | "f">;
```


## NonNullable<Type>

Type 类型中移除 `null` 和 `undefined`

### 源码

```ts
type NonNullable<T> = T extends null | undefined ? never : T;
```

### 例
// T0 = string | number
```ts
type T0 = NonNullable<string | number | undefined>;
```

## Parameters<Type>
Type 是函数类型，返回函数的参数。

### 源码

```ts
type Parameters<T extends (...args: any) => any> = T extends (...args: infer P) => any ? P : never;
```

### 例

```ts
declare function f1(arg: { a: number, b: string }): void

type T0 = Parameters<() => string>;  // []

type T1 = Parameters<(s: string) => void>;  // [string]

type T2 = Parameters<(<T>(arg: T) => T)>;  // [unknown]

type T4 = Parameters<typeof f1>;  // [{ a: number, b: string }]

type T5 = Parameters<any>;  // unknown[]

type T6 = Parameters<never>;  // never
```

## ReturnType<Type>
Type 是函数类型，返回 Type 的返回值

### 源码

```ts
type ReturnType<T extends (...args: any) => any> = T extends (...args: any) => infer R ? R : any;
```

### 例

```ts
declare function f1(): { a: number; b: string };
// T0 = string
type T0 = ReturnType<() => string>;
// T1 = void
type T1 = ReturnType<(s: string) => void>;
// T2 = unknown
type T2 = ReturnType<<T>() => T>;
// T3 = number[]
type T3 = ReturnType<<T extends U, U extends number[]>() => T>;
// T4 = { a: number; b: string }
type T4 = ReturnType<typeof f1>;
```

## InstanceType<Type>
返回构造函数类型 `Type` 的实例类型

### 源码

```ts
type InstanceType<T extends new (...args: any) => any> = T extends new (...args: any) => infer R ? R : any;
```

### 例
```ts
class C {
  x = 0;
  y = 0;
}
// type T0 = C
type T0 = InstanceType<typeof C>;
// type T1 = any
type T1 = InstanceType<any>;
//  type T2 = never    
type T2 = InstanceType<never>;
// Type 'string' does not satisfy the constraint 'abstract new (...args: any) => any'. 
type T3 = InstanceType<string>;
```

## ThisParameterType<Type>
返回函数的 `this` 参数类型，如果不是 `this`，返回 `unknow` 类型。

### 源码

```ts
type ThisParameterType<T> = T extends (this: infer U, ...args: any[]) => any ? U : unknown;
```

### 例

```ts
function toHex(this: Number) {
  return this.toString(16);
}
// n:number
function numberToString(n: ThisParameterType<typeof toHex>) {
  return toHex.apply(n);
}

function add(a: number, b: number) {
  return a + b
}
// unknown
type addParamerType = ThisParameterType<typeof add>
```

## OmitThisParameter<Type>
如果一个函数有指定的 `this` 类型，那么返回一个不带 `this` 类型的函数类型，否则还是返回原来的函数。

### 源码

```ts
type OmitThisParameter<T> = unknown extends ThisParameterType<T> ? T : T extends (...args: infer A) => infer R ? (...args: A) => R : T;
```

### 例

```ts
function toHex(this: Number) {
  return this.toString(16);
}
// () => string
const fiveToHex: OmitThisParameter<typeof toHex> = toHex.bind(5);
```

## ThisType<Type>
`ThisType` 不会返回一个转换之后的类型，提供基于上下文的 `this` 类型。注意，需要开启 `--noImplicitThis` 特性。

### 源码

```ts
interface ThisType<T> { }
```

### 例

```ts
type ObjectDescriptor<D, M> = {
    data?: D;
    methods?: M & ThisType<D & M>;  // Type of 'this' in methods is D & M
}

function makeObject<D, M>(desc: ObjectDescriptor<D, M>): D & M {
    let data: object = desc.data || {};
    let methods: object = desc.methods || {};
    return { ...data, ...methods } as D & M;
}

let obj = makeObject({
    data: { x: 0, y: 0 },
    methods: {
        moveBy(dx: number, dy: number) {
            this.x += dx;  // Strongly typed this
            this.y += dy;  // Strongly typed this
        }
    }
});

obj.x = 10;
obj.y = 20;
obj.moveBy(5, 5);
```


## 参考
- [TypeScript: Documentation - Utility Types](https://www.typescriptlang.org/docs/handbook/utility-types.html)
- [TypeScript Utility Types 学习笔记及源码解析 - 知乎](https://zhuanlan.zhihu.com/p/120802610)
- [ThisType | 深入理解 TypeScript](https://jkchao.github.io/typescript-book-chinese/typings/thisType.html)
