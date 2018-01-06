---
layout: post
title: "应用系统设计上的一些事"
date: 2015-01-25 0:15:00 +0800
categories: [思考]
---

<!--markdown-->### 应用系统设计上的一些事

首先，对于常见的web应用，都会有如下的结构，请先不要拿各种分布式来说事，因为那是在特定用户量下的特定产物，从来没有任何一种技术是一蹴而就的。
那么，我们不谈论大的架构，而是如何用很trick的方式，高质量的实现业务的需求。  
**PS: 这里我希望强调的一点是，任何开源的组件其实也是对业务的一种支持，没有任何一种技术的实现是脱离实际的。**

![Web Architecture](http://he-blog.oss-cn-beijing.aliyuncs.com/architecture.bmp)


好了，对于系统设计，我将从如下几个方面来阐述

1. 系统的层次划分
2. 编码规范
3. 代码质量


##### 系统的层次划分

对于系统的层次结构，我会有如下的定义: 

* 用户获取的View Object: 如团购的团单、微博的Feed
* 底层存储的Persistent Object: 数据的底层存储结构
* 数据提供者
* 数据使用者
* AOP相关: Log记录、权限判断、登陆状态监测等（具体场景具体分析）

给予以上的资源，我们定义，所有的Resource，我们当且仅当`Client`跟我们要的时候才返还给客户端。也就是说，`客户端发出请求，服务端组织资源`。

下面看一个第三方接入的图:  
![Thrid-Part](http://he-blog.oss-cn-beijing.aliyuncs.com/third-part.bmp)

对于第三方接入，我们会按如下方式实现:

1. 构建基础的访问工具，如Apache HttpClient的ConnectionManager
2. 封装第三方API，去业务化，并控制诸如超实时间等
3. 业务逻辑实现，并根据业务场景做相关的特殊处理

在对第三方模块接入时，我们消除了耦合，这里具体指功能逻辑与业务逻辑的耦合。因为诸如`账号融合`，`第三方登陆`，`登出`是与业务或者说与客户端操作紧密关联的。因此基于上述模型的设计是合理的。

ok，回到对系统层次的定义上来看，采用View、Presistence相结合的目的在于接触业务关联上的耦合。

如新浪微博的Feed List会包含Feed Object以及Comment List，那么好的设计应该是，当客户端发起请求后，服务端动态的组织这些数据，而不是将这些数据融合在一起。当进行详情页后，应该对Feed实体与Comment List进行拆分，分为两个接口请求，这样既保证了系统的整洁又保证了效率与性能。

对于package的命名，通常为`controller`，`service`，`biz`，`data-provier`。其中service可以引用各个biz，service之间不允许交叉引用。这样做的一个好处是，当需要进行服务化时，将很容易进行模块的垂直拆分与水平拆分。

总结一下系统层次划分:

1. 系统结构应该简洁，明了
2. 好的设计应该是符合客观世界规律的
3. 好的设计是充满美感的

#### 编码规范

时刻记着，你写的代码是否可读，你的用词是否准确，请避免诸如`get`这种引起误会的词语。

推荐两个读物

[The Art Of Readable Code](http://book.douban.com/subject/5442971/)  
[Goole Java Style](https://google-styleguide.googlecode.com/svn/trunk/javaguide.html)

#### 代码质量

> 作为一名RD，应该对自己的代码负责。——东东

对于TDD（Test Driven Developement）我一直觉得很傻逼，但这个的唯一借鉴意义于我是，`会让我思考我写的代码是否是测试的，如果不可测试，那就是有问题的。`

对于代码质量

1. 有效的注释
2. 可测试的代码（代码的组织结构会更松耦合）
3. 有效的测试用例：`边界值，如 Integer.MAX_VALUE`，`异常值 null`

