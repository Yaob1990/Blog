---
title: 忘记该忘记的，记住该记住的 —— git filter-branch
date: 2020-12-01 22:00:00
categories: 工具
tags: git
img: ../../coverImages/git.jpg
typora-root-url: ../../images
---

Pro git 中把 `filter-branch` 称为 **核弹**，让人难以忘记，也充分说明了该命令的威力。

### 问题场景

1. 在你的开源仓库中提交了，一个数据库的配置文件。
2. 同事在仓库中把 `node_modules`目录提交到仓库中，并且这个目录还被后续更新过。

### 解决方案：

可以使用`filter-branch`实现上述需求。

从每一个提交移除一个文件：

```shell
git filter-branch --tree-filter 'rm -f passwords.txt' HEAD
```

从每一个提交移除一个目录：

```shell
git filter-branch --tree-filter 'rm -rf node_modules' HEAD
```

限制移除范围：

```shell
git filter-branch --tree-filter 'rm -rf node_modules' HEAD~30..HEAD
```

全局修改邮箱地址：

```shell
git filter-branch --commit-filter '
    if [ "$GIT_AUTHOR_EMAIL" = "schacon@localhost" ];
    then
      GIT_AUTHOR_NAME="Scott Chacon";
      GIT_AUTHOR_EMAIL="schacon@example.com";
      git commit-tree "$@";
		else
		fi' HEAD
```

### 详细参数：

- `--tree-filter `表示修改文件列表

- `--msg-filter `表示修改提交信息，原提交信息从标准输入读入，新提交信息输出到标准输出

- `--commit-filter` 针对提交信息进行修改

- `--prune-empty `表示如果修改后的提交为空则扔掉不要。

- `-f`是忽略备份。不加这个选项第二次运行这个命令时会出错，意思是 git 上次做了备份，现在再要运行的话得处理掉上次的备份。

- `--all`是针对所有的分支。

  [^https://git-scm.com/docs/git-filter-branch]:
      官方文档
      [^依云：初次使用 git 的“核弹级选项”：filter-branch]: https://blog.lilydjwg.me/2011/4/22/tried-the-nuclear-option-filter-branch-of-git-the-first-time.26331.html

```
title: 忘记该忘记的，记住该记住的 —— git filter-branch
date: 2020-12-01 22:00:00
categories: 工具
tags: git
img: ../../coverImages/git.jpg
typora-root-url: ../../images
```

Pro git 中把 `filter-branch` 称为 **核弹**，让人难以忘记，也充分说明了该命令的威力。

### 问题场景

1. 在你的开源仓库中提交了，一个数据库的配置文件。
2. 同事在仓库中把 `node_modules`目录提交到仓库中，并且这个目录还被后续更新过。

### 解决方案：

可以使用`filter-branch`实现上述需求。

从每一个提交移除一个文件：

```
git filter-branch --tree-filter 'rm -f passwords.txt' HEAD
git filter-branch --tree-filter 'rm -rf node_modules' HEAD
git filter-branch --tree-filter 'rm -rf node_modules' HEAD~30..HEAD
git filter-branch --commit-filter '
    if [ "$GIT_AUTHOR_EMAIL" = "schacon@localhost" ];
    then
      GIT_AUTHOR_NAME="Scott Chacon";
      GIT_AUTHOR_EMAIL="schacon@example.com";
      git commit-tree "$@";
    else
    fi' HEAD
```

- `--tree-filter`表示修改文件列表

- `--msg-filter`表示修改提交信息，原提交信息从标准输入读入，新提交信息输出到标准输出

- `--commit-filter` 针对提交信息进行修改

- `--prune-empty`表示如果修改后的提交为空则扔掉不要。

- `-f`是忽略备份。不加这个选项第二次运行这个命令时会出错，意思是 git 上次做了备份，现在再要运行的话得处理掉上次的备份。

- `--all`是针对所有的分支。

  [[**https://git-scm.com/docs/git-filter-branch**] ] 官方文档

  [[**依云：初次使用 git 的“核弹级选项”：filter-branch**] ] https://blog.lilydjwg.me/2011/4/22/tried-the-nuclear-option-filter-branch-of-git-the-first-time.26331.html

### 详细参数：

全局修改邮箱地址：

限制移除范围：

从每一个提交移除一个目录：

```
title: 忘记该忘记的，记住该记住的 —— git filter-branch
date: 2020-12-01 22:00:00
categories: 工具
tags: git
img: ../../coverImages/git.jpg
typora-root-url: ../../images
```

Pro git 中把 `filter-branch` 称为 **核弹**，让人难以忘记，也充分说明了该命令的威力。

### 问题场景

1. 在你的开源仓库中提交了，一个数据库的配置文件。
2. 同事在仓库中把 `node_modules`目录提交到仓库中，并且这个目录还被后续更新过。

### 解决方案：

可以使用`filter-branch`实现上述需求。

从每一个提交移除一个文件：

```
git filter-branch --tree-filter 'rm -f passwords.txt' HEAD
git filter-branch --tree-filter 'rm -rf node_modules' HEAD
git filter-branch --tree-filter 'rm -rf node_modules' HEAD~30..HEAD
git filter-branch --commit-filter '
    if [ "$GIT_AUTHOR_EMAIL" = "schacon@localhost" ];
    then
      GIT_AUTHOR_NAME="Scott Chacon";
      GIT_AUTHOR_EMAIL="schacon@example.com";
      git commit-tree "$@";
    else
    fi' HEAD
```

- `--tree-filter`表示修改文件列表

- `--msg-filter`表示修改提交信息，原提交信息从标准输入读入，新提交信息输出到标准输出

- `--commit-filter` 针对提交信息进行修改

- `--prune-empty`表示如果修改后的提交为空则扔掉不要。

- `-f`是忽略备份。不加这个选项第二次运行这个命令时会出错，意思是 git 上次做了备份，现在再要运行的话得处理掉上次的备份。

- `--all`是针对所有的分支。

  [[**https://git-scm.com/docs/git-filter-branch**] ] 官方文档

  [[**依云：初次使用 git 的“核弹级选项”：filter-branch**] ] https://blog.lilydjwg.me/2011/4/22/tried-the-nuclear-option-filter-branch-of-git-the-first-time.26331.html

### 详细参数：

全局修改邮箱地址：

限制移除范围：

从每一个提交移除一个目录：

```
title: 忘记该忘记的，记住该记住的 —— git filter-branch
date: 2020-12-01 22:00:00
categories: 工具
tags: git
img: ../../coverImages/git.jpg
typora-root-url: ../../images
```

Pro git 中把 `filter-branch` 称为 **核弹**，让人难以忘记，也充分说明了该命令的威力。

### 问题场景

1. 在你的开源仓库中提交了，一个数据库的配置文件。
2. 同事在仓库中把 `node_modules`目录提交到仓库中，并且这个目录还被后续更新过。

### 解决方案：

可以使用`filter-branch`实现上述需求。

从每一个提交移除一个文件：

```
git filter-branch --tree-filter 'rm -f passwords.txt' HEAD
git filter-branch --tree-filter 'rm -rf node_modules' HEAD
git filter-branch --tree-filter 'rm -rf node_modules' HEAD~30..HEAD
git filter-branch --commit-filter '
    if [ "$GIT_AUTHOR_EMAIL" = "schacon@localhost" ];
    then
      GIT_AUTHOR_NAME="Scott Chacon";
      GIT_AUTHOR_EMAIL="schacon@example.com";
      git commit-tree "$@";
    else
    fi' HEAD
```

- `--tree-filter`表示修改文件列表

- `--msg-filter`表示修改提交信息，原提交信息从标准输入读入，新提交信息输出到标准输出

- `--commit-filter` 针对提交信息进行修改

- `--prune-empty`表示如果修改后的提交为空则扔掉不要。

- `-f`是忽略备份。不加这个选项第二次运行这个命令时会出错，意思是 git 上次做了备份，现在再要运行的话得处理掉上次的备份。

- `--all`是针对所有的分支。

  [[[**https://git-scm.com/docs/git-filter-branch**] ] ] 官方文档

  [[[**依云：初次使用 git 的“核弹级选项”：filter-branch**] ] ] https://blog.lilydjwg.me/2011/4/22/tried-the-nuclear-option-filter-branch-of-git-the-first-time.26331.html
