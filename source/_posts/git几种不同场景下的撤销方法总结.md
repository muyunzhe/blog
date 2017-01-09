---
title: git几种不同场景下的撤销方法总结
comments: true
categories: technology
tags: git
keywords: 'git'
date: 2016-12-30 18:08:49
---
本文主要介绍作为普通开发人员，在不需要特别深入了解git内部原理的情况下，如何能够对常用的基本操作灵活运用，操作熟练且不坑队友。
 <!--more-->
当然还是得简单了解下基本原理。
## Git架构
![git架构图](/img/git架构图.jpg)

### 目录结构
![git目录架构](/img/git目录架构.png)

### 四个阶段
 工作目录
 index(又称为暂存区)
 本地仓库
 远程仓库。
 ![git的四个阶段](/img/git的四个阶段.png)

### 存储原理
Git 是简单的 key-value 数据存储。它允许插入任意类型的内容，并会返回一个键值，通过该键值可以在任何时候再取出该内容。可以通过底层命令hash-object 来示范这点，传一些数据给该命令，它会将数据保存在 .git 目录并返回表示这些数据的键值。
如:
```
$ echo 'test content' | git hash-object w -stdin
d670460b4b4aece5915caf5c68d12f560a9fe3e4
```
参数 w 指示 hash-object 命令存储 (数据) 对象，若不指定这个参数该命令仅仅返回键值。-stdin 指定从标准输入设备 (stdin) 来读取内容，若不指定这个参数则需指定一个要存储的文件的路径。该命令输出长度为 40 个字符的校验和。这是个 SHA-1 哈希值──其值为要存储的数据加上你马上会了解到的一种头信息的校验和。现在可以查看到 Git 已经存储了数据：

```
$ find .git/objects -type f
.git/objects/d6/70460b4b4aece5915caf5c68d12f560a9fe3e4
```
可以在 objects 目录下看到一个文件。这便是 Git 存储数据内容的方式──为每份内容生成一个文件，取得该内容与头信息的 SHA-1 校验和，创建以该校验和前两个字符为名称的子目录，并以 (校验和) 剩下 38 个字符为文件命名 (保存至子目录下)。
通过 cat-file 命令可以将数据内容取回。该命令是查看 Git 对象的瑞士军刀。传入 -p 参数可以让该命令输出数据内容的类型：
```
$ git cat-file -p d670460b4b4aece5915caf5c68d12f560a9fe3e4
test content
```

## 基本命令
### git init or git clone
略

### git fetch命令
基本命令
`git fetch <远程主机名> <分支名>`

Git fetch:拉取而不合并
这一步其实是执行了两个关键操作:
- 创建并更新所有远程分支的本地远程分支.
- 设定当前分支的FETCH_HEAD为远程服务器的master分支 (上面说的第一种情况)

如果没有显式的指定远程分支, 则远程分支的master将作为默认的FETCH_HEAD.如果指定了远程分支, 就将这个远程分支作为FETCH_HEAD.
需要注意的是: 和push不同, fetch会自动获取远程`新加入'的分支.

示例一：git fetch origin branch1
设定当前分支的 FETCH_HEAD' 为远程服务器的branch1分支`.
注意: 在这种情况下, 不会在本地创建本地远程分支, 这是因为:这个操作是git pull origin branch1的第一步, 而对应的pull操作,并不会在本地创建新的branch.

示例二：git fetch origin branch1:branch2
只要明白了上面的含义, 这个就很简单了,
首先执行上面的fetch操作
使用远程branch1分支在本地创建branch2(但不会切换到该分支),
如果本地不存在branch2分支, 则会自动创建一个新的branch2分支,
如果本地存在branch2分支, 并且是`fast forward', 则自动合并两个分支, 否则, 会阻止以上操作.
git fetch origin :branch2
等价于: git fetch origin master:branch2

### git 关联远程分支
基本命令
```
git checkout -b [local-branch-name] [remote-name]/[remote-branch-name]
git checkout --track [remote-name]/[remote-branch-name]
git checkout -b [local-branch-name] --track [remote-name]/[remote-branch-name]
git branch --set-upstream master origin/next
```

### git pull
基本命令
`git pull <远程主机名> <远程分支名>:<本地分支名>`

如果合并需要采用rebase模式，可以使用--rebase选项。
`git pull --rebase <远程主机名> <远程分支名>:<本地分支名>`

如果远程主机删除了某个分支，默认情况下，git pull 不会在拉取远程分支的时候，删除对应的本地分支。这是为了防止，由于其他人操作了远程主机，导致git pull不知不觉删除了本地分支。
但是，你可以改变这个行为，加上参数 -p 就会在本地删除远程已经删除的分支。
```
$ git pull -p
# 等同于下面的命令
$ git fetch --prune origin
$ git fetch -p
```

### git push
基本命令
`git push <远程主机名> <本地分支名>:<远程分支名>`

如果省略远程分支名，则表示将本地分支推送与之存在"追踪关系"的远程分支（通常两者同名），如果该远程分支不存在，则会被新建。
`$ git push origin master`
上面命令表示，将本地的master分支推送到origin主机的master分支。如果后者不存在，则会被新建。

如果省略本地分支名，则表示删除指定的远程分支，因为这等同于推送一个空的本地分支到远程分支。
```
$ git push origin :master
# 等同于
$ git push origin --delete master
```
上面命令表示删除origin主机的master分支。

如果远程主机的版本比本地版本更新，推送时Git会报错，要求先在本地做git pull合并差异，然后再推送到远程主机。这时，如果你一定要推送，可以使用--force选项。
`$ git push --force origin`
上面命令使用--force选项，结果导致远程主机上更新的版本被覆盖。除非你很确定要这样做，否则应该尽量避免使用--force选项。

最后，git push不会推送标签（tag），除非使用--tags选项。
`$ git push origin --tags`

## Git的几种撤销方法
### 没add：只撤销某个文件的修改
```
$ git checkout filename
或者
$ git checkout -- filename
```
原理：git checkout将工作目录（working directory）里的文件修改成先前Git已知的状态。你可以提供一个期待回退分支的名字或者一个确切的SHA码，Git也会默认检出HEAD------即：当前分支的上一次提交。

注意：用这种方法“撤销”的修改都将真正的消失。它们永远不会被提交。因此Git不能恢复它们。此时，一定要明确自己在做什么！（或许可以用git diff来确定）

### add之后但没commit之前：撤回本次add
`git reset HEAD filename`

### Commit之后：
情况一：只是在最后的提交信息中敲错了字或者忘了有个文件没加进去
```
git commit -amend
或
git commit -amend -m "Fixes bug #42"
```

原理：
git commit --amend将使用一个包含了刚刚错误提交所有变更的新提交，来更新并替换这个错误提交。由于没有staged的提交，所以实际上这个提交只是重写了先前的提交信息。

例子：
```
$ git commit -m 'initial commit'
$ git add forgotten_file
$ git commit --amend
```

结果：最终只有一次提交，第二次提交修改了第一次提交

情况二、你已经在本地做了一些提交（还没push），但所有的东西都糟糕透了，你想撤销最近的三次提交
```
git reset
或
git reset --hard
```

原理：git reset将你的仓库纪录一直回退到指定的最后一个SHA代表的提交，那些提交就像从未发生过一样。

注意：默认情况下，git reset会保留工作目录（working directory）。这些提交虽然消失了，但是内容还在磁盘上。这是最安全的做法，但通常情况是：你想使用一个命令来“撤销”所有提交和本地修改-----那么请使用-hard参数吧。

### Push之后：
`git revert`

原理：git revert将根据给定SHA的相反值，创建一个新的提交。如果旧提交是“matter”，那么新的提交就是“anti-matter”-----旧提交中所有已移除的东西将会被添加进到新提交中，旧提交中增加的东西将在新提交中移除。这是Git最安全、也是最简单的“撤销”场景，因为这样不会修改历史记录-----你现在可以git push下刚刚revert之后的提交来纠正错误了。

## 几个特殊场景：
场景一：你已经提交了一些内容，并使用git reset --hard撤销了这些更改（见上面），突然意识到：你想还原这些修改！

使用撤销命令：git reflog和git reset, 或者git checkout

发生了什么：git reflog是一个用来恢复项目历史记录的好办法。你可以通过git reflog恢复几乎任何已提交的内容。


场景二：你提交了一些变更，然后你意识到你正在master分支上，但你期望的是在feature分支上执行这些提交。

使用撤销命令：
git branch feature, git reset --hard origin/master, 和 git checkout feature

发生了什么：你可能用的是git checkout b来建立新的分支，这是创建和检出分支的便捷方法-----但实际你并不想立刻切换分支。git branch feature会建立一个叫feature的分支，这个分支指向你最近的提交，但是你还停留在master分支上。
git reset --hard将master回退至origin/master，并忽略所有新提交。别担心，那些提交都还保留在feature上。最后，git checkout将分支切换到feature，这个分支原封不动的保留了你最近的所有工作。

## git diff
![git_diff](/img/git_diff.jpeg)

## 合并
### merge
![merge](/img/merge.jpeg)

### rebase
![rebase](/img/rebase.jpeg)

### cherry-pick
![cherry_pick](/img/cherry_pick.jpeg)

## 总结
![all](/img/all.jpg)
