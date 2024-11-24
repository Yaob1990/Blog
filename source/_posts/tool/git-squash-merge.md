---
title: git merge --squash
categories:
  - tool
tags: git
img: ../../coverImages/git.jpg
date: 2021-07-07 08:00:00
---

在开发过程中，git squash merge 是个比较低频使用的命令，这也是一个比较危险的命令（操作 git 记录），如果你不明白他的真实用途，建议不要使用。

## 场景

如果你在gitlab合并分支提交的时候勾选了`squash`选项，那么你的多个git提交记录会被合成一个提交记录，默认的提交名称变为`Merge branch 'branch-d' into 'master'`
![](/images/16256150683432.jpg)

## 结果
![](/images/16256149942897.jpg)

* 你的代码确实被合入了主分支，但是commit记录全丢
* 如果你继续用本地分支提交代码，你会发现，我自己和自己冲突了，what fuck ？？？？

## 原因
`git squash` 就是用来改写提交记录，压缩commit的，这就是它本来的用法鸭~，错的不是git，错的是你。。。。你不该选择 git merge --squash 。

master 视角
![](/images/16256154828571.jpg)

branch e 视角
![](/images/16256156817818.jpg)


上图 `Branch e` 被合入 `master` 分支，`Branch e` 有多个提交，被压缩成一个提交。这个提交是一个全新的提交，和 `Branch e` 没有任何关系，hash 也完全不同。所以现在`master`和`Branch e` 是两条平行线。这就解释了为啥自己和自己冲突，因为master，确实多了commit记录，但是因为 hash 完全不同，没办法和其他分支做关联。如果需要出现交汇点，你需要从 master 再 merge 到 `branch e`。

是不是觉得`git`太不人性化了，太蠢了？em......
