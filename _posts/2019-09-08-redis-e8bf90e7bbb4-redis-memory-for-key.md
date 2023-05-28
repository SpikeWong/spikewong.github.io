---
id: 231
title: 'Redis 运维: redis-memory-for-key'
date: '2019-09-08T13:25:10+08:00'
author: Spike
layout: post
guid: 'http://box5781.temp.domains/~spikeooo/?p=231'
permalink: /2019/09/08/redis-%e8%bf%90%e7%bb%b4-redis-memory-for-key/
views:
    - '392'
post_views_count:
    - '762'
image: /wp-content/uploads/2019/09/redis-150x150.png
categories:
    - 未分类
---

目前我们的业务线使用的单词查询服务必须通过 grpc 调用别人的接口，为了获得更快的响应速度。我通过一次批量的 grpc 调用在 redis 缓存了一份单词的全量拷贝（以 hashkey 的形式）。但是 key 太大的话也会带来一些问题，比如大量的 qps 打到该 key 所在的节点导致集群倾斜，为了了解目前这个 key 占用了多少内存，我用 redis-memory-for-key 进行了查询，下面我就来讲讲如何使用 redis-memory-for-key.

#### 安装 rdb-tools

redis-memory-for-key 是 rdb-tools 中的一个 script,所以首先我们需要通过 pip 安装 rdb-tools.

```bash
pip install rdbtools 
```

<div class="wp-block-image"><figure class="aligncenter">![](/assets/wp-content/uploads/2019/09/安装rdbtools-2.png)</figure></div>#### 向 redis 插入 key 

```bash
HMSET runoobkey name "redis tutorial" description "redis basic commands for caching" likes 20 visitors 2
```

<figure class="wp-block-image">![](/assets/wp-content/uploads/2019/09/向-redis-插入-key.png)</figure>####  使用 redis-memory-for-key 

```bash
redis-memory-for-key -s localhost -p 6379 runoobkey
```

<figure class="wp-block-image">![](/assets/wp-content/uploads/2019/09/使用-redis-memory-for-key.png)</figure>由图中可以看出 elements 的个数为 4 ,占用的内存为 158 bytes.