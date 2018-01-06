---
layout: post
title: "关于web项目的层次结构的思考"
date: 2015-04-08 23:19:00 +0800
categories: [思考]
---

在传统的MVC模型下，会对项目进行分层，层次结构为`controller-service-dao`。但随着项目的复杂，该结构会有所演进。

通常会表现为：

第一阶段：

* service: `service.interface`，`service.impl`

这种方式，本质上，是将通用的interface暴露给了外界，让各个service进行了交叉引用，并有了一定的解耦含义。

同时可能的一种场景为，某个Controller会引用，大于等于1个的service，来进行操作。这种方式的问题在于：

1. 交叉引用导致的业务脉络复杂化
2. 在服务化层面的接耦困难
3. 代码冗余、维护成本高

第二阶段：

由于第一阶段产生的若干问题，我们自顶向下的考虑项目结构，来自`前端(移动、PC)`的请求通过`Controller`传递给`Service`，那么一个相对合理的模型为每一个Controller与一个Service进行配对。

e.g. UserController <-> UserService

这样，保证了Service的独立性，那么接下来，对资源的共享的情况如何解决嘞？在这里我们通常采用的方式是引入`Biz层`。

对Biz层的定义为：资源的管理者。同时参见下图

![Web Architecture](http://he-blog.oss-cn-beijing.aliyuncs.com/Web-Project-Structure.bmp)

下面简单描述下各个层面的职责，当然这个部分也是仁者见仁智者见智的：

* Controller
    * 承接来自客户端的请求
    * 参数有效性校验
    * 调用Service获取相关result资源
    * 构建Resp，如json、html模版等
* Service
    * 处理业务逻辑
    * 通过Biz完成`持久化资源`到`视图层资源`的封装
* Biz
    * 资源数据的管理器，CRUD（增删改查）
    * 整合不同的资源提供者，来自Cache、DB、RPC调用等

基于上述的描述，基本可以解决第一阶段的3点问题，代码逻辑更清晰透明。

第三阶段：

这个阶段，会涉及到一些相对抽象的主题，诸如，服务分级等。

这个主题不准备在该篇中展开，日后会结合实际的例子来加以阐述。

