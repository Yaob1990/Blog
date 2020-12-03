---
title: 弱水三千，只取一瓢 —— git cherry-pick
date: 2020-12-03 23:00:00
categories: 工具
tags: git
img: ../../coverImages/git.jpg
---

### 场景

双主线的模式 AB，独自演进，存在一个共同的 bug，在 dev 上面修改好 bug 之后，需要合并到 AB 中，并且 A、B、dev 是一直在演进的，不能直接合并入 AB。

### 解决

```shell
git checkout A // 切换到 A 分支
git cherry-pick C1 // 拣选 C1 并合并到 A 分支
git checkout B  // 切换到 B 分支
git cherry-pick C1  // 拣选 C1 并合并到 B 分支
```

`cherry-pick` 捡樱桃，这名字很可爱：）。commit 就像一地的樱桃，我们需要做的就是去捡起大大小小的樱桃。

```
$ git cherry-pick <commitHash>
```

获得需要合并的 commit 值，切换到待合并的分支，执行 `cherry-pick`，

### 详细参数

    --quit                      终止反转或拣选操作
    --continue                  继续反转或拣选操作
    --abort                     取消反转或拣选操作
    --skip                      跳过当前提交并继续
    --cleanup <模式>             设置如何删除提交说明里的空格和#注释
    -n, --no-commit             不要自动提交
    -e, --edit                  编辑提交说明
    -s, --signoff               添加 Signed-off-by: 签名
    -m, --mainline              <父编号>选择主干父提交编号
    --rerere-autoupdate         如果可能，重用冲突解决更新索引
    --strategy                  <策略> 合并策略
    -X, --strategy-option       <选项> 合并策略的选项
    -S, --gpg-sign[=<key-id>]   GPG 提交签名
    -x                          追加提交名称
    --ff                        允许快进式
    --allow-empty               保留初始化的空提交
    --allow-empty-message       允许提交说明为空
    --keep-redundant-commits    保持多余的、空的提交

### 尾

红尘过往，弱水三千，只取一瓢。
git 的 `cherry-pick` 是简单明确的，不会出错。而你我的生活，从没有平行宇宙，这一瓢水又从何而取...
