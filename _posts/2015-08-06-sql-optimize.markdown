---
layout: post
title: "记一次SQL优化的历程"
date: 2015-08-06 11:37:00 +0800
categoryies: jekyll update
---

### 记一次SQL优化的历程

> 背景，由于线上业务的访问量增大，直透redis进入到DB的操作也变多。因此针对此场景，进行一些必要调整。

本文，大抵组织如下结构:

1. 简要数据量介绍
2. 介绍两种优化方式：延迟写入（业务向）与创建索引（DB向）
3. 描述一个错误的分业方式
4. 优化后的数据对比

#### 简单数据量介绍

先描述一些数据量，DB的访问量约为`30w`，其中`100ms`以上约为`1w+`次，其中`write`操作占据了`90%`以上。单表最大约为`200w`条记录左右。

需要表明的是，30w的访问量，对mysql来说so easy，只可能是操作方式不当导致的。

#### 延迟写入

接下来，举个栗子，来说明下问题。

在业务系统中通常会有一个叫做`用户行为`的模块，其目的，在于记录用户的每一次操作，并基于这个操作进行一些定量的分析。

PS：大公司会将这个东西拆出来，仅记录LOG日志，然后交由`数据组`处理即可。

那么，就有一种简单的做法，当用户每次`invoke`了某个接口，就在DB中insert一条记录。

上述这个过程，在并发量大的时候，也就是会产生[锁等待](http://dev.mysql.com/doc/refman/5.6/en/internal-locking.html)。

这个在早期的社区类系统中也很常见，例如热帖的访问计数、点赞计数等等。诸如，`update topic_count set view_count = view_count + 1 where topic_id = xxx`这样语句在并发下就会有问题。

那么解决的办法有之（并不一定最优）

通过`延迟写入`来优化，我们可以设置一个`15分钟的bucket`，将15分钟内的操作进行汇总，然后统一更新，这样一次db的batch操作即可完成Write操作。

如果我们每15分钟的Write为一次task，为了避免任务失败而导致的数据丢失，可以考虑使用`task queue`，从queue中peek一个task，成功在pop出来，否则add到队尾。

Ok，这次SQL优化主要是通过`延迟写入`与`db索引`搞定的。

#### 创建索引

下面说说，db索引，更详细的部分可以参考

索引大致有三种

* primary index
* column indexes
* multiple-column indexes

同时，对于mysql的操作类型，<、<=、=、>=、>、in、like在合理的操作下都可以使用索引来进行加速。

下面特别说明下组合索引（multiple-column index）有一些`最左前缀`的要求。

e.g. 假设创建了组合索引 (col1, col2, col3)

那么对于where子句下的如下情况可以走索引

* (col1)
* (col1, col2)
* (col1, col2, col3)

以上两点，即可满足`大部分基础业务`调优了。

#### 一个错误的分页手段

e.g: SELECT nid FROM news WHERE is_delete = 0 AND audit_state != 3 AND sid = 1001 union all SELECT nid FROM manual_news WHERE is_delete = 0 AND sid = 1001 ORDER BY nid desc limit 1000, 20

问题在于，limit m, n 会查询 (m + n)条记录后再返回top n

解决办法：用where子句后的条件过滤掉大量的数据集，例如 where nid > 1000 limit 10这样就会很快

#### 优化后的数据对比

前后大致经多了一周多的时间，来做这件事情，通过了大致三方面结合的手法`读写分离`、`业务调整（延迟写入）`、`索引构建`。

最终，在`60w`次的调用下，超过100ms的数量为3k左右，占整体比例的0.5%

