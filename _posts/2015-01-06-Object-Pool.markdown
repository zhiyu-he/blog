---
layout: post
title: "Object Pool引子"
date: 2015-01-06 01:04:00 +0800
categoryies: [common]
---

### Object Pool 引子？

> 引用资料包括

[设计模式解析(第二版)](http://book.douban.com/subject/5360960/)


##### 对于Object Pool的唠叨
Object Pool被译为`对象池`模型。在我们实际的使用中其标准化的实现触手可及。
如`thread pool`，`connection pool(jdbc、redis...)`等等。

同时`Object pool`本身在处理`共享资源`以及对该资源的`单点联系`时较为有益。

因为，对于共享资源来讲，池子中的Object被所有能看到他的人所共享，但同时，每个Object在某一时刻又仅能被一个人所使用。

##### Object Pool的一些特征

###### 意图
在创建对象比较昂贵，或者对于特定类型能够创建对象树木有限制时，管理对象的重用。

###### 问题
1. 对象的`创建`与`管理`必须遵循一组定义明确的规则集。
2. 对象的`使用规则`与`重用规则`。
3. `错误的处理`。

