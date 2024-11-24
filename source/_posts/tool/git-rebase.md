---
title: 重置过去，面向未来 —— git rebase
categories:
  - tool
tags: git
img: ../../coverImages/git.jpg
typora-root-url: ../../images
date: 2020-11-30 07:18:46
---

> git rebase 是一个危险的操作，如果不能熟练掌握，请不要使用它。（它并不是不可或缺的）



`git rebase` 是一个平常使用较少的一个命令，这次准备分享 git ，把一系列不常用的 git 命令，都熟悉一下。简单明了，不深究原理，只面向解决问题。

### 场景一：合并多次提交

本地开发，存在多次提交记录，需要合并提交，方便查看。

```shell
commit 36bf7ff90f1cf33991cc68dac8e078c3f6e14ca3 (HEAD -> master)
Date:   Mon Nov 30 21:17:25 2020 +0800

    add 2.txt

commit 1569f1e8dfa9bca26e0233780a43e0961209ad8c
Date:   Mon Nov 30 21:17:12 2020 +0800

    add 1.txt
```

1. 列出本地最近记录

   ````shell
   git rebase -i HEAD~2
   ````

2. 这时候会进入 `vi`模式

   ```shell
   pick 60b798f add a b
   pick d39bbcd add 1.txt add 2.txt
   
   # 变基 8a749eb..d39bbcd 到 8a749eb（2 个提交）
   #
   # 命令:
   # p, pick <提交> = 使用提交
   # r, reword <提交> = 使用提交，但修改提交说明
   # e, edit <提交> = 使用提交，进入 shell 以便进行提交修补
   # s, squash <提交> = 使用提交，但融合到前一个提交
   # f, fixup <提交> = 类似于 "squash"，但丢弃提交说明日志
   # x, exec <命令> = 使用 shell 运行命令（此行剩余部分）
   # b, break = 在此处停止（使用 'git rebase --continue' 继续变基）
   # d, drop <提交> = 删除提交
   # l, label <label> = 为当前 HEAD 打上标记
   # t, reset <label> = 重置 HEAD 到该标记
   # m, merge [-C <commit> | -c <commit>] <label> [# <oneline>]
   # .       创建一个合并提交，并使用原始的合并提交说明（如果没有指定
   # .       原始提交，使用注释部分的 oneline 作为提交说明）。使用
   # .       -c <提交> 可以编辑提交说明。
   #
   # 可以对这些行重新排序，将从上至下执行。
   ```

   vi 模式修改 `pick` 保存之后，就可以实现合并提交记录。

   **注意：**第一行`pick 60b798f add a b`不能是`s`，可以是`pick`或者`reword`。合并必须有一个基点，第一行就是一个基点，后续的变更合并都必须有第一行这个可合并的对象存在。

3. 如果退出了 `vi`窗口，想要继续编辑：

   ```shell
   git rebase --edit-todo
   ```

### 场景二：分支合并

`git merge` 和 `git rebase` 都可以实现分支合并。

git merge,合并之后，存在多个分支线。

![image-20201130215051796](/../../../../../Library/Application Support/typora-user-images/image-20201130215051796.png)

git rebase 合并之后，也就是上图的 `develop add 3.txt`并不会产生新的合并记录，详细过程如下：

1. 首先，`git` 会把 `develop` 分支里面的每个 `commit` 取消掉；
2. 其次，把上面的操作临时保存成 `patch` 文件，存在 `.git/rebase` 目录下；
3. 然后，把 `develop` 分支更新到最新的 `master` 分支；
4. 最后，把上面保存的 `patch` 文件应用到 `develop` 分支上；

 这也就是 `git rebase`为什么被称为 **变基 **的原因，随着主分支的发展，变基操作，只是同步了主分支的更改，在最新的主分支上面进行开发。有点类似 `git stash`, 先保存内容，等分支同步之后，再释放内容。



### 警告

因为变基操作会直接修改历史记录，千万不要在公共分支上面使用变基操作，你应该只在你自己的分支上进行`git rebase`操作。
