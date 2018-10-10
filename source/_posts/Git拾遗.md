---
title: "Git拾遗"
date: 2018-10-10 21:00:00
tags: [programming, Git]
categories: programming
---

### 序言
最近做项目的时候要把几个repo合在一个repo里，而且要保留所有的history，这才意识到自己只会最基本的`Git`操作，对`Git`的实现原理了解得不够。花了几小时学习了一下`Git`，并记录在此文中。

### Git与其他版本控制系统的区别
+ `Git`是分布式，每个本地都保存一份文件与所有修改。
+ 其他版本控制系统为中央集中式，中央保存所有修改，本地只有一份快照，如果要切换到其他version，需要与中央交互。
+ `SVN`追踪文件的变化，而`Git`的版本控制模型基于快照。比如说，一个`SVN`提交由仓库中原文件相比的差异（diff）组成。而`Git`在每次提交中记录文件的 完整内容。这让很多`Git`操作比`SVN`来的快得多，因为文件的某个版本不需要通过版本间的差异组装得到——每个文件完整的修改能立刻从`Git`的内部数据库中得到。

<img src="http://omcdckn46.bkt.clouddn.com/git-vcs-structure.png" width="500px">

<img src="http://omcdckn46.bkt.clouddn.com/git-vcs-checkin.png" width="500px">

### 三个状态 Working Directory - Stage - Head
+ `Working Dir`: 本地工作目录
+ `Stage`: 缓存区（不然就只能都commit了）
+ `HEAD`: （一般）指向最新一次commit的引用

<img src="http://omcdckn46.bkt.clouddn.com/three-states.png" width="500px">

### Git 一些命令
#### `git init <dir>`与`git init —bare <dir>.git`
+ `git init <dir>` 新建一个`dir`并`init`一个repo，`git clone`前不需要，会直接在当前目录`init`
+ `git init —bare <dir>.git` 新建一个`bare repo`，**不含** working tree，一般用于中央服务器上（想想remote确实是.git结尾的）
+ 如果本地`git init —bare a.git`了，就可以`git clone a.git`到别的目录，对文件进行修改，然后`git push`到`a.git`里
+ 不能直接在`bare repo`里修改文件（一般你也找不到），本地repo用来commit，remote的用来push

#### `git revert` / `git reset` / `git clean`

* `git revert <commit>` 回滚**某一次**commit，*提交一个新的commit*来取消这次的修改，会多一个commit，而且会留在记录里，比如发现一个bug是某一次commit引起的，来revert这次commit来修复这个bug，也适用于push到remote

* `git reset <commit>`: 回滚到某一个commit，同时回滚n个commit，但会保留本地的修改。当使用`git reset —hard <commit>`时，会删除本地的修改到那个commit时的状态，如果不指定commit，则为最近一次提交
* `git reset <file>`: 把一个添加到缓存区的文件恢复到未缓存区，修改仍然保留; `git reset`所有文件放回未缓存区
* `git clean`：清理没有追踪的文件。`git reset —hard HEAD`会清理本地目录，把追踪文件的修改给reset，但新增加的文件依旧在，这时需要`git clean -f`来取消这些文件的添加 `-df`则为目录和文件

```bash
git reset --hard HEAD
git clean -df  // 新添加的目录和文件
# 让本地工作目录回到上一次commit的状态
```
#### `git commit —amend`
+ 将缓存区的修改和最新的一次commit合并成一个新的commit，替换了最新的commit
+ `git commit --amend --no-edit` 不修改上次commit的信息

#### `git rebase <basebranch>`
+ rebase就是说把base分支改成别的分支，也就是说本分支上做的改动在那个新的base上重做一次
+ [这篇文章](https://git-scm.com/book/zh/v2/Git-%E5%88%86%E6%94%AF-%E5%8F%98%E5%9F%BA) 关于rebase讲得很好
```bash
git checkout experiment
git rebase master
# master作为base，共同父亲c2到自己做的修改在master上重做一遍，做完还是在experiment上哟
```

<img src="http://omcdckn46.bkt.clouddn.com/rebase.png" width="500px">

### Git 内部实现
#### Data Model
+ blob：用来存储文件内容，每次改变一个文件就形成新的blob
+ tree：解决了文件名-sha的对应问题，每个tree指向一系列blob和tree，可以把tree想成是一个directory
+ commit：用于储存commit信息，指向具体的tree节点

<img src="http://omcdckn46.bkt.clouddn.com/git-data-model.png" width="500px">

#### Branching
+ 每个branch在git实现里就是一个指向last commit的文件，因为从last commit开始按照方向只能有一个history

<img src="http://omcdckn46.bkt.clouddn.com/git-branching.png" width="400px">

```bash
cat .git/refs/heads/master
c641e4f0d19df0570667977edff860fed8f6c05a
```

+ `.git/HEAD`记录了当前分支的信息，不然git就不知道当前在什么branch了， HEAD其实是指向一个ref的文件

<img src="http://omcdckn46.bkt.clouddn.com/git-head.png" width="500px">

```bash
cat .git/HEAD
ref: refs/heads/feature
```

#### Index
+ index是个文件，里面记录了working directory/staging/repository的文件变化，当`git add`后，只是更新这些文件的变化

<img src="http://omcdckn46.bkt.clouddn.com/git-states-change.png" width="500px">

##### 实例讲解Index
+ 当`git checkout feature`的时候，发生的事情：
    + HEAD移到feature
    + 把这个commit中包含的信息更新到index（index文件只存信息，不是存blob）
        + mtime：文件最后修改时间
        + wdir：当前working directory的文件版本
        + stage：当前staging上的文件版本
        + repo：当前repo的文件版本
    + 把working directory更新到这个commit所指向的文件内容

<img src="http://omcdckn46.bkt.clouddn.com/git-index-cb.png" width="500px">

+ 当对`index.php`进行修改后，working dir中的文件mtime就发生了变化，此刻如果跑`git status`，wdir和mtime就发生了变化，这个命令会发现wdir和stage内容不同，就会告诉你这个文件进行了修改，但还没有stage

<img src="http://omcdckn46.bkt.clouddn.com/git-index.png" width="500px">

```bash
On branch feature
Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git checkout -- <file>..." to discard changes in working directory)
modified:   index.php
no changes added to commit (use "git add" and/or "git commit -a")
```

+ 当`git add index.php`后，stage的值也会变成wdir的值，而且会为新文件创建blob文件

<img src="http://omcdckn46.bkt.clouddn.com/git-index-2.png" width="500px">

+ 当`git commit`后:
    + 会创建一个新的commit和tree object
    + 把feature指向新的commit
    + index中repo的值也会更新

<img src="http://omcdckn46.bkt.clouddn.com/git-index-3.png" width="500px">

### 关于合并repo的解决方案
+ `git filter-branch`用于对之前所有的记录进行修改，比如`git filter-branch --tree-filter 'rm -f passwords.txt'` HEAD会对每一次提交都先删除一个文件；`git filter-branch --commit-filter ....` HEAD又会对所有的commit进行遍历然后做修改
```bash
git filter-branch --index-filter '
        git ls-files -s |
        sed "s,\t,&'"$dir"'/," |
        GIT_INDEX_FILE="$GIT_INDEX_FILE.new" git update-index --index-info &&
        mv "$GIT_INDEX_FILE.new" "$GIT_INDEX_FILE"
    ' HEAD
```