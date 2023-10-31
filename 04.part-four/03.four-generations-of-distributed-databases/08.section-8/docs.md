---
title: 番外篇
taxonomy:
    category: docs
---

说完了四代分布式数据库的变迁，我们再来讨论一些周边的重要概念。

### Shared-Nothing、Shared-Memory 和 Shared-Disk

Shared-Nothing 只是一种思想，并不是一种明确的数据库架构，它非常笼统，只是描述了一种状态。在这里我们简单讨论一下 Shared-Nothing。

Shared-Nothing 描述的是一种分布式数据库的运行状态：两台物理机，除了网络通信之外，不进行任何资源共享，CPU、内存、磁盘都是独立的。这样，整个系统的理论性能就可以达到单机的二倍。

怎么理解 Shared-Nothing 思想呢？把它和 Shared-Disk 放到一起就明白了：

Shared-Disk：多台机器通过共享 SAN 磁盘的方式协同工作，让系统整体性能突破单机的极限。Oracle RAC 是这个架构的佼佼者，不过它的成功并不在于磁盘，而在于它的分布式锁（CACHE FUSION）：RAC 利用时间戳和分布式锁实现了分布式事务和多台机器同时可写，大幅提升了集群的性能。注意，时间戳在这里又出现了。CACHE FUSION 其实已经可以被称作 Shared-Memory 了。对这个话题感兴趣的读者可以自己了解，我们不再深入。

21 世纪初，Oracle 推出了 Shared-Disk 的 RAC，IBM 推出了 Shared-Nothing 的 DB2 ICE。十年后，Oracle RAC 发展的如火如荼，而 DB2 ICE 已经消失在了历史的长河中。

但是，2012 年 Google 发布了 Spanner 论文，在非常成熟的世界上最大规模的 KV 数据库之上，构建 SQL 层，实现持久化、事务和多版本并发控制，扛起了 Shared-Nothing 技术方向的大旗，直到今天。

### MongoDB 小故事

十年前笔者在新浪云（SAE）实习的时候，听过一个关于 MongoDB 的技术小故事：当时，SAE 的 KV 服务是使用 MongoDB 实现的，在规模大到一定程度以后，性能会突然下降，SAE 自己解决不了这个问题，就给 MongoDB 开发组的各位大哥买机票请他们到北京理想国际大厦 17 层现场来帮忙，研究了几天，MongoDB 开发组的人说：你们换技术吧，MongoDB 解决不了你们这个规模的问题，然后 SAE 的 KV 就更换技术方案来实现了。

### DBA 晕倒砸烂花盆

也是在 SAE，笔者坐在厕所附近临过道的工位（上厕所很方便），某天早上刚上班，亲眼看到 SAE 的一名 MySQL DBA 从厕所里出来后，晕倒在笔者面前，砸烂了一个大花盆。数据库作为系统架构中最重要的那个单点的残酷，可见一斑。

### 列存储思想

与其将列存储认定为数据库的一种，笔者倒是觉得它更应该被称作一种思想：观察数据到底是被如何读取，并加以针对性地优化。

列存储有点像第一性原理在数据库领域的应用：不被现实世界所束缚，没有屈服于 B 树和它亲戚们的淫威，勇敢地向更底层看去，思考着在我们大量读取数据时，数据怎样组织才能读的更快。

在读取一行数据时，显然 B+ 树的效率无人能及，但是当我们需要读取 100 万行数据中的某一列时，B+ 树就需要把这 100 万行数据全部载入内存：每次将一页 16KB 载入内存，循环这一页内的 14 行数据，把这个特定的字段复制出来；重复执行这个操作 71429 次，才能得到我们想要的结果。这显然是 B+ 树非常不擅长的需求。

而列存储将数据基于行的排布翻转过来了：所有数据基于列，致密地排列在磁盘上，这样对某一列的读取就变成了磁盘顺序读，即便是机械磁盘，顺序读也非常快。

#### 列存储数据库 Clickhouse 堪称俄罗斯人暴力美学的典范，和 Nginx 的气质很像

Clickhouse 推荐使用尽量多的 CPU 核心，对单核性能无要求，笔者使用 Xeon E5-V2 旧服务器测过，速度确实非常惊人，8000 万行的表，查询起来不仅比 MySQL 快，比 Hadoop 也快特别多。