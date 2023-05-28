---
id: 468
title: '数据库的 max_connections 真的越大越好吗?'
date: '2020-07-26T20:18:36+08:00'
author: Spike
layout: post
guid: 'http://box5781.temp.domains/~spikeooo/?p=468'
permalink: /2020/07/26/db-max_connections-%e7%9c%9f%e7%9a%84%e8%b6%8a%e5%a4%a7%e8%b6%8a%e5%a5%bd%e5%90%97/
post_views_count:
    - '2591'
image: /wp-content/uploads/2019/04/mysql-150x150.jpg
categories:
    - Mysql
tags:
    - 数据库
---

## 概览

最近接手的项目有很多让我觉得匪夷所思的地方，其中就包括一个 8 cores,16 GB 内存的 postgresql 9.4 版本的 RDS 设置了 1500 的 max\_connections。之前我对 max\_connections 到底该设置多少也没有一个具体的概念，但是 1500 这么大的数目让我不得不去了解一下 max\_connections 到底该如何设置？  
熟悉数据库的朋友都知道 max\_connections 会影响数据库的并发连接数，当数据库到达设置的 max\_connections 数值时，则会拒接新的 client。在[mysql的基础架构是怎样的呢？](/assets/2019/04/26/mysql%e7%9a%84%e5%9f%ba%e7%a1%80%e6%9e%b6%e6%9e%84%e6%98%af%e6%80%8e%e6%a0%b7%e7%9a%84%e5%91%a2%ef%bc%9f/ "mysql的基础架构是怎样的呢？")我们就提到过 mysql server 的连接器，而 max\_connections 就是约束了连接器。这么看来，如果把 max\_connections 设置为 infinite 好像能够提高数据库的并发度，但是事实却并不是这样。接下来，就让我们一起来看看 max\_connections 是如何影响并发度并且怎样设置一个合理的大小呢?

## max\_connections 是如何影响并发度的?

拿 mysql 举例，当我们设置 max\_connections 值为1时，此时包括 mysql 为 super user 保留的 connection，mysql 此时能够允许 client 的连接个数为 2。  
为了了解 mysql 在达到 max\_connections 后是如何处理新的数据库连接的，首先确保我们这个时候同时打开了 3 个 terminal,其中两个 terminal 通过 `mysql -u root -p` 可以连接成功，当第三个 terminal 尝试连接时就会出现 `ERROR 1040 (HY000): Too many connections`。

![](/assets/wp-content/uploads/2020/07/too_many_connections-1024x845.jpg)

postgresql 也类似于此，当我们达到 postgresql 达到的 limit 时，则会出现 `psql: error: could not connect to server: FATAL:  remaining connection slots are reserved for non-replication superuser connections`。

 因此，由此我们可以知道 max\_conenctions 限制了当前数据库最大的连接数量，这也就限制了我们应用的最大并发度。这听上去好像我们应该尽可能地增大 max\_connections ，防止因为 max\_connections 的数值过小而没有充分利用数据库的性能，降低了整体并发度。但是受限于服务器的资源限制，max\_connections 并不能无限制地增长。并且在设置了过大的 max\_connections 情况下，数据库会因为保持了大量的连接而使服务器资源耗尽而变得无法响应。

## max\_connections 为什么不是越大越好?

在讨论为什么 max\_connections 不是越大越好之前，我们需要了解数据库是如何维护和管理客户端连接的。线程和进程在计算机科学中是永远绕不开的话题，进程作为资源管理的基本单位，能够为应用提供隔离的资源空间。而线程则是比进程更小的能够独立运行的单位，因为维护线程所需的资源更小，所以线程具有比进程更大的并发支持。我们平时所执行的各种各样的程序或指令，都是由 cpu 调度进程或者线程来完成的，进程或线程中保存了执行任务所需要的指令和数据。

而我们每新建一条 connection 时，数据库就 fork 一个进程或者创建一个线程来维护当前 connection 的运行指令和上下文信息。

下面我们以 postgresql 为例，用 pgbench 测试 postgresql 在不同的活跃连接数下的性能。关于 pgbench 的使用说明你可以访问 http://www.postgres.cn/docs/9.3/pgbench.html 。

我们将在 3.1 Ghz, 2 cores 8GB 的机器上使用 pgbench 测试 postgresql 在不同活跃连接数下的性能表现。然后我们通过更新 postgresql.conf 将 max\_connections 设置为 1500，并确保在更新 conf 文件后重启 postgresql。

![](/assets/wp-content/uploads/2020/07/tps随connections变化-1024x803.jpg)

<center>tps 随 connections 变化</center>  
![](/assets/wp-content/uploads/2020/07/average_latency-1024x802.jpg)  
<center>average\_latency(ms) 随 connections 变化</center>由上图可以看出，在随着 connnections 数量不断增加的情况下，tps 先上升然后下降，而 average\_latency 则一直在增加。

为什么结果是这样的呢？

- 1.postgesql 会给每个 connection 都 fork 一个子进程，为了维护每个 connection 的上下文信息需要内存资源，当 connections 数量太多的时候就可能会使服务器过载，此时的服务器性能就会急剧下降。mysql 虽然不会为每个 connection 都创建一个进程，但是 mysql 也会为每个 connection 分配一个线程，线程虽然相较于进程所消耗的资源较少，但是也同样会存在过载的问题。
- 2.一个 core 同一时间只能执行一个进程或线程，所谓并发只是通过将 cpu 时间分片，然后分配给不同的线程，由于 cpu 在线程切换时需要重新加载对应线程的上下文信息，所以在这种机制下，单核串行执行任务（不考虑 IO 等待）一定比并发执行的时间短，当 connections 数量不断增加时，cpu 不断在各个 connections 的查询中切换，这将使 latency 增加。

从上面的第二点原因我们可能会又会产生另一个疑问，既然串行执行的效率更高，那么为什么不设计成 max\_connection 为1呢?这是因为我们的数据库会经常进行 IO 的系统调用和锁等待，当前进程或者线程在等待 IO 或者等待锁释放时，此时 cpu 是闲置的，所以为了提高 cpu 的使用效率，我们一定程度上允许并发执行的查询数 > 系统的 cores 数，特别是 IO 密集型计算更是如此。

至此，我们发现，设置较大的 max\_connections 值时，可能会导致数据库因接受了太多的连接而性能下降，甚至因为资源耗尽而无法响应的风险。而较小的 max\_conenctions 值又可能会成为并发的瓶颈。

## 那 max\_connections 设置为多少合适呢?

没有银弹！！在考虑 max\_connections 的设置数量时，往往要根据业务场景，业务并发连接数以及服务器器负载等综合考量。单就业务场景来说，我们可以将业务分为长连接业务与短连接业务。考虑除 connections pool 外，如果 max\_conenctions 设置较小，那么单条记录的插入更新等语句很可能会被 DDL 等大事务长连接语句阻塞。而较大的 max\_connections 则又可能导致服务器资源耗尽。  
在这里，我建议根据实际的应用场景以及服务器条件进行测试来设置 max\_connections 个数，同时应用连接池，连接池内的同一个 connection 能够在不同的客户端连接之间复用，从而避免了为每个客户端连接单独维护进程或线程的开销。同时在到达连接池的最大连接数后，连接池也不会拒绝新的客户端连接，新的客户端连接会阻塞等待连接池释放空闲的连接。  
同时你也可以参考 mysql 官方建议

> In the case of active connections using temporary/memory tables, memory usage can go even higher. On systems with small RAM or with a hard number of connections control on the application side, we can use small max\_connections values like 100-300. Systems with 16G RAM or higher max\_connections=1000 is a good idea, and per-connection buffer should have good/default values while on some systems we can see up to 8k max connections, but such systems usually became down in case of load spikes.

如果你用的是 postgresql 则可以参考 [](https://pgtune.leopard.in.ua/)<https://pgtune.leopard.in.ua/>\#/

> 本文参考文章  
> [How to Benchmark PostgreSQL Performance](https://severalnines.com/blog/benchmarking-postgresql-performance)  
> [MySQL Connection Handling and Scaling](https://mysqlserverteam.com/mysql-connection-handling-and-scaling/)  
> [pgbench](https://www.postgresql.org/docs/10/pgbench.html)  
> [MySQL Error: Too many connections](https://www.percona.com/blog/2013/11/28/mysql-error-too-many-connections/)