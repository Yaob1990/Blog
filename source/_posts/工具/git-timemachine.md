---
title: 回到过去的一万种方式 —— git 时光机
date: 2020-12-02 23:00:00
categories: 工具
tags: git
img: ../../coverImages/git.jpg
typora-root-url: ../../images
---

> 时光无法倒流，git 却让我们有机会修改历史

查看修改历史，代码回退是开发中经常用到的命令，但是很多时候，我们并不是非常明确其中的区别。这篇博客尝试说清楚其中的区别。

### git checkout

```shell
git checkout hotfix // 切换分支
git checkout abbcde // 切换 commit
```

`checkout` 能够返回 commit ，用于查看 commit ，但是并不能修改或者回退 commit。

### git revert

```shell
git revert HEAD^ // 回退到上一次的提交
```

`revert` 的回退严格来说并不是回退，他是用之前的 commit ，复制一遍，产生新的 commit ，实现的代码回退。

### git reset

```shell
git reset c1 // 回退commit,保留源码到工作区
git reset --soft c1 // 回退到某个版本，只回退了commit的信息，不会恢复到index file一级。如果还要提交，直接commit即可；
git reset --mixed c1 // 回退到某个版本，回退文件保存在工作区中，并且是 unstaged 状态，如果需要提交，需要 staged，之后再commit
git reset --hard c1 // 彻底回退到某个版本，本地的源码也会变为上一个版本的内容，撤销的commit中所包含的更改被冲掉；
```

### 一万种方式？？

一生二二生三三生万物，施主，不要有执念。
