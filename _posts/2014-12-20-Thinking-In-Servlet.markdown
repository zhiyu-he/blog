---
layout: post
title: "Thinking In Servlet Part One"
date: 2014-12-20 23:51:00 +0800
categories: [Java, servlet]
---

### Thinking in Servlet（一）

> 引用资料包括  

1. [JSR 154: JavaTM Servlet 2.4 Specification](https://jcp.org/en/jsr/detail?id=154)  
2. [Java Servlet技术简介](http://www.ibm.com/developerworks/cn/education/java/j-intserv/j-intserv.html)

#### 1 Servlet小品

##### 1.1 Servlet是什么？

servlet是一枚web组件，由`servlet container`管理。同时，servlet通过`request/response`与web client进行交互。

##### 1.2 Servlet Continer是什么？

servlet container是web-server或application-server的一部分。

同时servlet container会处理（预处理？）:

1. 来自客户端的request、response
2. 根据MIME-based解码request
3. 根据MIME-based格式化resopnse
4. servlet container会管理servlet的生命周期

servlet container需要支持标准协议:

1. HTTP
    * HTTP/1.0
    * HTTP/1.1
2. HTTPS

##### 1.3 当一枚客户端的请求遇到了Servlet Container

1. 客户端发起一枚 HTTP Request
2. Web Server收到了请求，并转交给了`Servlet Container`
    * Servlet Container 与 Web Server可能在同一进程、也可能不在
3. Servlet Continer 决定调用哪个servlet，并由这个servlet处理request/response
4. servlet通过`request object`处理具体的业务逻辑
    * 通过request可以获得
        * remote user
        * REQUEST METHOD: GET、 POST、 PUT、 DELETE
        * 数据信息
    * 生成response data
5. 确定servlet完成后，将控制权返回给`web server`?

##### 1.4 Servlet的优势

* They are generally much faster than CGI scripts `because a different process model is used`.

#### 2. Servlet的生命周期

##### 2.1 加载与实例化

1. servlet的加载与实例化，由servlet container负责，初始化的时机为:
    * servlet container 启动时
    * `延迟加载`，当相关servlet需要处理request时

2. servlet class 加载的位置
    * 本地文件系统
    * 远程文件系统
    * 网络service

##### 2.2 初始化

1. 调用`init`方法进行资源初始化
    * 在现在MVC的框架中，通常不会在`init`方法中初始化db或其他资源，目的在于解耦。

2. 初始化过程中的Error
    * 不会将该Servlet变为活动状态

##### 2.3 请求处理

1. 调用`service`方法
    * Request Object -> ServletRequest、ServletResponse
    * 对于HTTP请求 -> HttpServletRequest、HttpServletResponse

2. 多线程问题
    * servlet container 会在同一时间调用`service`方法多次
    * 保证servlet的`service`方法的`无状态性`
    * 将servlet的`service`方法视为一个dispatcher

3. 处理过程中的Exception
    * UnavailableException: servlet contatiner 会从service中移除该servlet，同时调用`destroy method`
    * 临时性的，则返回503

4. 线程安全问题（没懂）
    

##### 2.4 Service结束

servlet的生命周期由`servlet container`控制，当`servlet container`决定移除servlet时，将会调用其`destory`method，调用场景
    
 * 回收内存
 * 容器关闭

在调用`destory`method之前，将会等待所有在service中执行的thread执行完毕，或等待其超时。

`destory`method之后后，jvm的gc将回收该servlet。


#### 3. 小结

Servlet是Java Web的重要组成部分，不夸张的说，Servlet是很多web框架的根基。对servlet的本质有一个清晰的认识是及其有必要的。

如前文所述，主要描述以下几个方面

1. servlet的生存环境
2. servlet的生命周期
3. servlet的本质含义
