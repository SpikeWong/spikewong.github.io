---
id: 452
title: 'Deep dive into git &#8211; 存储原理'
date: '2020-06-07T16:22:51+08:00'
author: Spike
layout: post
guid: 'http://box5781.temp.domains/~spikeooo/?p=452'
permalink: /2020/06/07/deep-dive-into-git-%e5%ad%98%e5%82%a8%e5%8e%9f%e7%90%86/
post_views_count:
    - '576'
image: /wp-content/uploads/2020/06/git.jpeg
categories:
    - git
tags:
    - Git
---

## 前言

前段时间我的博客后台 css 一直处于 not work 的状态，导致我一直没法更新文章。后来还是找到法子解决了，如果有类似问题的朋友可以参考这个链接🔗<https://stackoverflow.com/questions/18769141/wordpress-admin-not-loading-css-js>  
![](https://spike.dev/wp-content/uploads/2020/06/wordpress-博客-css-失效-1024x542.png)  
今天我就跟大家分享一下近期学习 git 底层的一些知识，包括 git 的 3 种对主要象，git merge 的方式以及原理，常用的 git 命令等等。

## Git 简介

git 是 linux 之父林纳斯开发的一个版本控制工具，最初只是用于 linux 内核开发项目，话说在 git 开发之前其实内核项目已经在使用 BitKeeper ，但是碍于 BitKeeper 是商业软件，具有开源精神的大佬们就一直颇有不满了。某天组内一个叫[安德鲁](https://zh.wikipedia.org/wiki/%E5%AE%89%E5%BE%B7%E9%AD%AF%C2%B7%E5%9E%82%E9%B3%A9 "安德鲁")的大佬 hack 了一下 BitKeeper，于是 BitKeeper 厂商就觉得安德鲁这群人可能要逆向我的软件了，就终止了与 linux 小组的合作。林纳斯被逼上梁山后，最终花了 10 天开发出了初代 git。  
话说在如今，为什么 git 作为一个版本控制工具会一家独大呢？首先版本控制工具可以分为集中式版本控制工具和分布式版本控制工具。版本控制工具需要解决的一个很重要的问题是多人如何协同开发，换句话说多个人对同一个文件的修改采取什么策略合并？集中式版本控制系统给出的方案是决定权由单点控制，当有人对某个个文件进行修改时就对文件进行加锁，其余人则被禁止对文件进行变更，这就是我们常见的悲观锁！而分布式版本控制系统则允许多个人同时对本地文件进行修改，在工作流合并的时候根据合并策略对文件内容进行选择性合并。Git 作为分布式版本控制工具的代表提倡 Everthing is local，你不必连接版本控制服务器就能在本地进行开发。  
除此以外，git 有别于其他版本控制工具每次提交只是记录文件的变更记录，git 每次提交的时候都会为当前仓库生成整个快照。那么有个小伙伴可能就有疑问了，如果像 oracle 这样大的项目，我只是更改了某行代码的注释，难道 git 也要把整个项目快照下来，这样岂不是很浪费空间？为了解决这个问题，git 在每次提交的时候都会为未做任何变更的文件保存一个链向上一个版本的指针。所以，整个 git 的快照流程将会如下图所示

![](https://spike.dev/wp-content/uploads/2020/06/git-snapshots.png)

## Git 存储原理

话说前面我们讲了 git 对于整个项目是通过快照的方式存储的，那么对于单个的文件 git 是采用什么方式存储的呢?在详细讲解之前，先跟我来了解一下关于文件存储相关的知识。git 底层是类似键值存储的内容存储系统，何为内容存储系统呢？大家熟悉的磁盘介质等等物理存储系统在同一个地址空间可以存放电影、文本等多种内容格式。但内容存储系统的存储地址则会根据待存内容发生改变。大家一定熟悉每次 commit 时 git 返回给我们的 40 位 16 进制数吧(`86e1eb34c394cda21aee2d7ff950c04098f21a27`)。这串数字就唯一标志了我们对应的 commit。git 根据提交的内容以及一些头部信息进行一次 sha-1 计算就得到了我们这串40位的字符串。git 除了能通过这串字符串找到文件对应的存储位置，还能够确保文件自提交后未发生任何修改，一旦修改，这串40位16进制数就会发生变更。在提交之后，我们就可以在 .git 目录下看到对应的对象

```shell
spike:git_practice apple1$ find .git/objects -type f
.git/objects/86/e1eb34c394cda21aee2d7ff950c04098f21a27
//其中 40 位数字的前 2 位作为目录，后 38 位作为文件地址
```

这 40 位数字具体生成的算法是怎样的呢？不急，请听我慢慢道来！  
首先我们先向 git 中写入一个 blob 对象。git 告诉我们这个对象的存储地址为 `6bf0c97a7f84620a0bb4cf6380ec307748e043bd`。

```shell
spike:git_practice apple1$ echo -n 'w'|git hash-object -w --stdin
6bf0c97a7f84620a0bb4cf6380ec307748e043bd
```

其实，git 根据我们输入的 'w' 字符生成了 `blob 1\0w`，其中 blob 是存储内容的对象种类，除了 blob 还有 commit 和 tree。数字1是指内容的长度，\\0是固定的空字符，最后就是需要存储的内容了。通过对上述内容进行一次 sha-1 计算就可以得到最终我们想要的40位16进制数了。

```shell
spike:git_practice apple1$ echo -n 'w'|git hash-object -w --stdin
6bf0c97a7f84620a0bb4cf6380ec307748e043bd
spike:git_practice apple1$ printf 'blob 1\0w'|sha1sum
6bf0c97a7f84620a0bb4cf6380ec307748e043bd  -
```

除此以外，git 还会对 `blob 1\0w` 这样的存储内容进行一次 zlib 压缩。所以我们直接用 cat 命令查看对应文件是不能正常编码的，如果想查看对应文件内容需要用到`git cat-file -p `命令

```
spike:git_practice apple1$ cat .git/objects/6b/f0c97a7f84620a0bb4cf6380ec307748e043bd
xK??OR0d(
         h
spike:git_practice apple1$ git cat-file -p 6bf0c97a7f84620a0bb4cf6380ec307748e043bd
w
```

除了 blob 对象（对应所有文件的存储），git 还有 commit 和 tree 对象。  
tree 对象是简化版的 unix 文件系统，它能解决文件的命名以及将多个文件组织在一起的问题。我们可以通过 `git cat-file -p master^{tree}` 查看 master 分支上最新的一个 tree 对象的相关信息。

```shell
spike:git_practice apple1$ git cat-file -p master^{tree}
100644 blob 8a705fb93f3bc9e47c6c13e79dc5c61bcdcc2885    my.txt
// 可以看到该 tree 对象指向了一个 blob 对象
```

我们也可以通过 `write-tree`命令为当前暂存区生成一个 tree 对象。

```
spike:git_practice apple1$ git status
位于分支 rebase
要提交的变更：
  （使用 "git restore --staged <文件>..." 以取消暂存）
    修改：     my.txt

spike:git_practice apple1$ git write-tree
0398da6ea3e139197041403fd41da4121d1a5c33
spike:git_practice apple1$ git cat-file -p 0398da6ea3e139197041403fd41da4121d1a5c33
100644 blob d2fc3401f9e101ed69000ead02a63d1e8000b421    my.txt
```

最后我们要介绍的是 commit 对象，这也是我们最熟悉的一个对象了。前面我们讲过 git 每一次 commit 的时候都会对当前文件系统进行一次快照，那么 git 到底是怎样实现的呢？那就让我们来看看 commit 对象里面保存了哪些信息。

```shell
spike:git_practice apple1$ git log
commit 86e1eb34c394cda21aee2d7ff950c04098f21a27 (HEAD -> rebase, master)
Author: anoymouscoder <hello@spike.wiki>
Date:   Wed Jun 3 21:13:03 2020 +0800

    first commit
spike:git_practice apple1$ git cat-file -p 86e1eb34c394cda21aee2d7ff950c04098f21a27
tree a9dc5348d362b100960d487c4f31e45397cbe7a1
author anoymouscoder <hello@spike.wiki> 1591189983 +0800
committer anoymouscoder <hello@spike.wiki> 1591189983 +0800

first commit
```

我们可以看到 commit 对象里保存了 author, commiter 以及一个 tree 对象（非项目的首个 commit 还会有 parent 信息）。所以当我们每次生成 commit 的时候，git 会记录下 commiter 等信息，还会链向一个 tree，这个 tree 里面就包含了我们当前项目里的所有内容 。所以至此我们就能明白整个 git 的文件存储链路是  
![](https://spike.dev/wp-content/uploads/2020/06/git-objects-存储.png)

## 总结

在本节，我们知道了 git 底层是一个根据键值存储的内容文件系统，每次 commit git 都会对当前系统生成一份快照。其中 git 对需要保存的文件（即 blob 对象）进行 sha-1 计算得出存储地址，再通过类 unix 文件系统的 tree 对象对文件进行组织。而我们的 commit 对象则保存了提交者以及 tree 等相关信息。  
在下一节，我们将了解到 git merge 原理的相关知识。