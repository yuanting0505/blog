---
title: "Git拾遗"
date: 2018-10-10 21:00:00
tags: [programming, Git]
categories: programming
---

### 序言

最近做项目的时候要把几个 repo 合在一个 repo 里，而且要保留所有的 history，这才意识到自己只会最基本的`Git`操作，对`Git`的实现原理了解得不够。花了几小时学习了一下`Git`，并记录在此文中。

### Git 与其他版本控制系统的区别

-   `Git`是分布式，每个本地都保存一份文件与所有修改。
-   其他版本控制系统为中央集中式，中央保存所有修改，本地只有一份快照，如果要切换到其他 version，需要与中央交互。
-   `SVN`追踪文件的变化，而`Git`的版本控制模型基于快照。比如说，一个`SVN`提交由仓库中原文件相比的差异（diff）组成。而`Git`在每次提交中记录文件的 完整内容。这让很多`Git`操作比`SVN`来的快得多，因为文件的某个版本不需要通过版本间的差异组装得到——每个文件完整的修改能立刻从`Git`的内部数据库中得到。

<img src="https://s1.ax1x.com/2018/11/22/FPPi9A.png" class="non-full" width="500px">

<img src="https://s1.ax1x.com/2018/11/22/FPP9tH.png" class="non-full" width="500px">

### 三个状态 Working Directory - Stage - Head

-   `Working Dir`: 本地工作目录
-   `Stage`: 缓存区（不然就只能都 commit 了）
-   `HEAD`: （一般）指向最新一次 commit 的引用

<img src="https://s1.ax1x.com/2018/11/22/FPPkct.png" class="non-full" width="500px">

### Git 一些命令

#### `git init <dir>`与`git init —bare <dir>.git`

-   `git init <dir>` 新建一个`dir`并`init`一个 repo，`git clone`前不需要，会直接在当前目录`init`
-   `git init —bare <dir>.git` 新建一个`bare repo`，**不含** working tree，一般用于中央服务器上（想想 remote 确实是.git 结尾的）
-   如果本地`git init —bare a.git`了，就可以`git clone a.git`到别的目录，对文件进行修改，然后`git push`到`a.git`里
-   不能直接在`bare repo`里修改文件（一般你也找不到），本地 repo 用来 commit，remote 的用来 push

#### `git revert` / `git reset` / `git clean`

-   `git revert <commit>` 回滚**某一次**commit，*提交一个新的 commit*来取消这次的修改，会多一个 commit，而且会留在记录里，比如发现一个 bug 是某一次 commit 引起的，来 revert 这次 commit 来修复这个 bug，也适用于 push 到 remote

-   `git reset <commit>`: 回滚到某一个 commit，同时回滚 n 个 commit，但会保留本地的修改。当使用`git reset —hard <commit>`时，会删除本地的修改到那个 commit 时的状态，如果不指定 commit，则为最近一次提交
-   `git reset <file>`: 把一个添加到缓存区的文件恢复到未缓存区，修改仍然保留; `git reset`所有文件放回未缓存区
-   `git clean`：清理没有追踪的文件。`git reset —hard HEAD`会清理本地目录，把追踪文件的修改给 reset，但新增加的文件依旧在，这时需要`git clean -f`来取消这些文件的添加 `-df`则为目录和文件

```bash
git reset --hard HEAD
git clean -df  // 新添加的目录和文件
# 让本地工作目录回到上一次commit的状态
```

#### `git commit —amend`

-   将缓存区的修改和最新的一次 commit 合并成一个新的 commit，替换了最新的 commit
-   `git commit --amend --no-edit` 不修改上次 commit 的信息

#### `git rebase <basebranch>`

-   rebase 就是说把 base 分支改成别的分支，也就是说本分支上做的改动在那个新的 base 上重做一次
-   [这篇文章](https://git-scm.com/book/zh/v2/Git-%E5%88%86%E6%94%AF-%E5%8F%98%E5%9F%BA) 关于 rebase 讲得很好

```bash
git checkout experiment
git rebase master
# master作为base，共同父亲c2到自己做的修改在master上重做一遍，做完还是在experiment上哟
```

<img src="https://s1.ax1x.com/2018/11/22/FPPF1I.png" class="non-full" width="600px">

### Git 内部实现

#### Data Model

-   blob：用来存储文件内容，每次改变一个文件就形成新的 blob
-   tree：解决了文件名-sha 的对应问题，每个 tree 指向一系列 blob 和 tree，可以把 tree 想成是一个 directory
-   commit：用于储存 commit 信息，指向具体的 tree 节点

<img src="https://s1.ax1x.com/2018/11/22/FPCO6x.png" class="non-full" width="500px">

#### Branching

-   每个 branch 在 git 实现里就是一个指向 last commit 的文件，因为从 last commit 开始按照方向只能有一个 history

<img src="https://s1.ax1x.com/2018/11/22/FPPl3n.png" class="non-full" width="400px">

```bash
cat .git/refs/heads/master
c641e4f0d19df0570667977edff860fed8f6c05a
```

-   `.git/HEAD`记录了当前分支的信息，不然 git 就不知道当前在什么 branch 了， HEAD 其实是指向一个 ref 的文件

<img src="https://s1.ax1x.com/2018/11/22/FPCqpR.png" class="non-full" width="500px">

```bash
cat .git/HEAD
ref: refs/heads/feature
```

#### Index

-   index 是个文件，里面记录了 working directory/staging/repository 的文件变化，当`git add`后，只是更新这些文件的变化

<img src="https://s1.ax1x.com/2018/11/22/FPCvnK.png" class="non-full" width="600px">

##### 实例讲解 Index

-   当`git checkout feature`的时候，发生的事情：
    -   HEAD 移到 feature
    -   把这个 commit 中包含的信息更新到 index（index 文件只存信息，不是存 blob）
        -   mtime：文件最后修改时间
        -   wdir：当前 working directory 的文件版本
        -   stage：当前 staging 上的文件版本
        -   repo：当前 repo 的文件版本
    -   把 working directory 更新到这个 commit 所指向的文件内容

<img src="https://s1.ax1x.com/2018/11/22/FPCXX6.png" class="non-full" width="600px">

-   当对`index.php`进行修改后，working dir 中的文件 mtime 就发生了变化，此刻如果跑`git status`，wdir 和 mtime 就发生了变化，这个命令会发现 wdir 和 stage 内容不同，就会告诉你这个文件进行了修改，但还没有 stage

<img src="https://s1.ax1x.com/2018/11/22/FPCz7D.png" class="non-full" width="600px">

```bash
On branch feature
Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git checkout -- <file>..." to discard changes in working directory)
modified:   index.php
no changes added to commit (use "git add" and/or "git commit -a")
```

-   当`git add index.php`后，stage 的值也会变成 wdir 的值，而且会为新文件创建 blob 文件

<img src="https://s1.ax1x.com/2018/11/22/FPCL11.png" class="non-full" width="500px">

-   当`git commit`后:
    -   会创建一个新的 commit 和 tree object
    -   把 feature 指向新的 commit
    -   index 中 repo 的值也会更新

<img src="https://s1.ax1x.com/2018/11/22/FPCx0O.png" class="non-full" width="500px">

### 关于合并 repo 的解决方案

-   `git filter-branch`用于对之前所有的记录进行修改，比如`git filter-branch --tree-filter 'rm -f passwords.txt'` HEAD 会对每一次提交都先删除一个文件；`git filter-branch --commit-filter ....` HEAD 又会对所有的 commit 进行遍历然后做修改

```bash
git filter-branch --index-filter '
        git ls-files -s |
        sed "s,\t,&'"$dir"'/," |
        GIT_INDEX_FILE="$GIT_INDEX_FILE.new" git update-index --index-info &&
        mv "$GIT_INDEX_FILE.new" "$GIT_INDEX_FILE"
    ' HEAD
```
