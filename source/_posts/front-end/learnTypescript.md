---
title: Typescript 该不该学？
categories:
  - front-end
img: ../../coverImages/typescriptLogo.jpg
tags:
  - Typescript
typora-root-url: ..\..
date: 2020-06-02 09:00:00
---





# 引

all in js，编程语言的相互编译并不是复杂的问题，在[github](https://github.com/jashkenas/coffeescript/wiki/List-of-languages-that-compile-to-JS)上面可以看到大量的编译器，`java`，`ruby`，`python`，甚至远古的 `smallTalk`，`lisp`，都可以编译为` javascript`。

`javascript` 的方言派系大致可以分成两派：

1. `CoffeeScript` ：由ruby社区创建的方言，`CoffeeScript` 吸收了 `Ruby`, `Python`, `Haskell` 语言的设计思想, 将 `JavaScript` 的代码变得清晰简洁, 盛极一时。
2. TypeScript 是 Microsoft 推出的编程语言, 项目由大名鼎鼎的 C# 首席架构师 Anders Hejlsberg 操刀. 它的特点是在 JavaScript 的基础上实现了强类型和静态类型检查, 加入了很多静态语言才有的概念。

`CoffeeScript `已然式微，Typescript 背靠微软这颗大树，在前端界掀起腥风血雨，知名的开源框架，`React`、`Vue`、`Angular`无不投入其怀抱，甚至`github`上面稍微用户多一点的类库，都会有 issue 提到是否提供 Typescript 的  `.d.ts` 文件（有这个文件，就提供了 `Typescript`的支持 ）。

官方说，`Typescript` 是`Javascript`的超集，似乎可以把他理解成基于 `Javascript` 的独立语言？我并不这么理解，`Typescript` 最终的执行宿主环境，还是v8引擎，它最终还是要编译成`JavaScript`。它的用户绝大部分也还是`JavaScript`用户，这些原因，使得它对新特性的添加非常克制，新特性几乎都可以在TC39（一个推动 JavaScript 发展的委员会，由各个主流浏览器厂商的代表构成，制定ECMAScript标准，标准生成的流程，并实现）中找到原型。

`Typescript` 只不过是 `Beta` 版本的 `Javascript`。学不学完全不是问题了，现在不学，以后肯定还是要学:smile:。

 

# 项目

### [一句 web端](https://word.aocoding.com/) （ 基于 `vue` 和 Typescript 写法）



```typescript
<script lang="ts">
// class 组件写法
import { Component, Vue } from 'vue-property-decorator'
import { Mutation } from 'vuex-class'
import { message } from 'ant-design-vue'

// 装饰器
@Component({})
export default class IndexPage extends Vue {
  // 原data
  form = {
    word: '',
    from: '',
    source: ''
  }

  // vuex state
 // ! 断言不为空，兼容写法（断言）
  //  由于Ts最新版本使用了strictPropertyInitialization，如果变量没有在构造函数中使用或赋值，都需要添加!，进行显式赋值断言
  @Mutation('showloading') showloading!: boolean

  rules = {
    word: [
      {
        required: true,
        message: '请输入内容',
        whitespace: true,
        trigger: 'blur'
      }
    ],
    from: [
      {
        required: true,
        message: '请输入来源',
        whitespace: true,
        trigger: 'blur'
      }
    ]
  }

// 兼容写法
// nuxt 中$api被挂载到了this下面
  $api: any

  // 兼容写法
  // formRef 类型
  // 解释：https://stackoverflow.com/questions/52109471/typescript-in-vue-property-validate-does-not-exist-on-type-vue-element
  get formRef(): Vue & { validate: () => boolean } {
    return this.$refs.form as Vue & { validate: () => boolean }
  }

  public async onSubmit() {
    const self = this
    const verify: boolean = await this.formRef.validate()
    if (verify) {
      try {
        const { code, msg, data } = await self.$api.index({
          ...this.form
        })
        if (code === 0) {
          message.success('添加成功')
          this.form = {
            word: '',
            from: '',
            source: ''
          }
        } else {
          message.error(msg)
        }
      } catch (error) {}
    }
  }
}
</script>
```

github:    https://github.com/justOneWord/one_word_web

个人感受：

1. 传统vue组件写法过度为`vue-class-component`写法，改变比较大，使用装饰器代替传统的代码组织形式。
2. 因为对Typescript的支持不完善，需要写部分兼容代码。
3. 前端的代码类库提供的类型定义文件不完善，很多类库都没有提供，部分表态在vue3之后提供。现阶段可能需要手动声明为any使用。





### 一句服务端 （基于 nest.js 和 typescript 写法）

`wordController`(部分)：

大量使用装饰器语法，简化写法

```typescript
// 装饰器
@Controller('word')
export class WordController {
  constructor(private readonly wordService:WordService) {
  }
 
  // 三个装饰器
  @Post()
  // 参数校验管道
  @UsePipes(ValidationPipe)
  // 手动返回200，否则会返回201，也可以放到全局拦截器去做
  @HttpCode(200)
  // body 类型 WordDto
  async add(@Body() body: WordDto) {
  	return   await this.wordService.add(body)
  }

  @Get('random')
  random (){
    return this.wordService.random()
  }
}

```

`wordService`（部分）:

```typescript
constructor(
    // 注入 redis
    private readonly redisService: RedisService,
    // 数据库注入
    @InjectRepository(Word) private readonly wordReposition: Repository<Word>,
    @InjectRepository(Hitoapi)
    private readonly hitoApiReposition: Repository<Hitoapi>,
  ) {}

 // 指定 body 类型为 `WordDto`
async add(body: WordDto) {
    // 查找是否存在
    // result 会自动推导出类型，不需要手动指定
    const result = await this.wordReposition.find({ word: body.word });
    if (result.length > 0) {
        // ApiCode 代码提示，也是Typescript提供的能力
      throw new ApiException('资源已经存在', ApiCode.EXIST_ERROR, 200);
    }

    const newWord = await this.wordReposition.create({
      ...body,
    });
    console.log('newWord');
    const newResult = await this.wordReposition.save(newWord);
    if (newResult) {
      //  插入成功
      return true;
    } else {
      throw new ApiException('添加失败', ApiCode.BUSINESS_ERROR, 200);
    }
  }

```

github: https://github.com/justOneWord/one_word_api

个人感受：

1. `nestjs` 框架基于 Typescript 开发，整体使用行云流水，没有为了用Typescript而用Typescript的感觉。
2. 代码层次清晰，具有完善的 mvc 结构，比`express`方便。



参考文档：

1. [漫谈 JavaScript 方言与派系](https://www.blackglory.me/javascript-dialects-and-factions/)

2. [前端开发的瓶颈与未来之路](https://zhuanlan.zhihu.com/p/139731168)