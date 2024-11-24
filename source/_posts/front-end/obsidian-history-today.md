---
categories:
  - front-end
title: obsidian-history-today 插件开发
date: 2024-11-24 22:20
---
重度 obsidian 用户，几乎每天一篇或多篇笔记，常见的人员管理也放到了上面。希望有个插件能够查看往年今日，于是有了本插件。

# 安装
obsidian 商店搜索 `History Today` 安装之前就可以使用了。

# 开发流程
仓库：https://github.com/Yaob1990/obsidian-history-today
使用的官方脚手架进行的开发，代码主要是 `cursor` 写的，我打打下手。

开发结束之后，提交到官方仓库：https://github.com/Yaob1990/obsidian-releases
先是程序自动审核，给我提了样式写死的问题，不想改，直接把这个功能删除了。

后续人工审核：https://github.com/obsidianmd/obsidian-releases/pull/4611#issuecomment-2477461352

![](/images/20241124221136.png)

不得不说，这老哥 review 是非常详细了，我上面不想改导致一些多余的文件，也都提出来了，得，咱安心修改吧。

提交之后，很快再次 review ，何入到主分支，商店就能搜索到了。

# 想法
1. ai 已经非常强了，能生成很多的业务逻辑
2. ai 知识库原因新的 api 往往不清楚，需要手动去调整，设置里面的自动建议就是这个原因
3. ob 审核流程比较长，尽量完善之后再提交