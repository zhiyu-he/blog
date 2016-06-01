---
layout: post
title: "Demo Of Paoding-Rose"
date: 2014-11-07 01:38:00 +0800
categoryies: jekyll update
---

### paoding-rose 环境构建

#### 准备工作

1. jdk 安装完成
2. gradle 安装完成
3. 网络ok
4. 构建项目结构工具[下载地址](https://github.com/townsfolk/gradle-templates)

#### 构建项目

1. 生成一枚空的web项目结构如下

   ```
   | |____src
   | | |____main
   | | | |____java
   | | | | |____org
   | | | | | |____demo
   | | | | | | |____controllers
   | | | | | | |____service
   | | | |____resources
   | | | |____webapp
   | | | | |____WEB-INF
   | | |____test
   | | | |____java
   | | | |____resources
   ```

2. 配置build.gradle文件

   ```
   apply plugin: 'jetty'
   apply plugin: 'war'     // web项目的打包格式
   apply plugin: 'idea'    // ide的支持，eclipse则填写eclipse
   war.archiveName = 'ROOT.war'
   group = 'cinao'
   version = '1.0.0'
   compileJava.options.encoding = 'UTF-8' // 设置默认字符编码为UTF-8
   ```
3. 配置web.xml文件

   ```
   <?xml version="1.0" encoding="utf-8"?>
   <beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
    http://www.springframework.org/schema/beans/spring-beans-2.5.xsd
    http://www.springframework.org/schema/context
    http://www.springframework.org/schema/context/spring-context-2.5.xsd"
       default-lazy-init="true">

   <context:annotation-config/>
   <context:component-scan base-package="org.demo"/>
   
   <bean id="jade.dataSource.org.demo.dao"
          class="org.apache.commons.dbcp.BasicDataSource"
          destroy-method="close">
        <property name="driverClassName" value="com.mysql.jdbc.Driver"/>
        <property name="url"
                  value="jdbc:mysql://127.0.0.1:3306/demo?useUnicode=false&amp;character_set_client=utf8mb4"/>
        <property name="username" value="root"/>
        <property name="password" value="root"/>
    </bean>
   
   </beans>
   ```
   
4. 填写相关demo代码

   ```
   package org.demo.controllers;

   import net.paoding.rose.web.annotation.Path;
   import net.paoding.rose.web.annotation.rest.Get;

   /**
    * @author hezhiyu on 14/11/6.
    */
   @Path("/")
   public class DemoController {
   
       @Get("hi")
       public String sayHi() {
          return "@" + "Hello World!";
       }
   }
   ```

#### 跑起来

1. 在项目目录下执行gradle war，当现实如下状态时，即打包成功

   ```
   ➜  paoding-rose-base git:(master) gradle war
   :compileJava
   :processResources
   :classes
   :war
   
   BUILD SUCCESSFUL
   
   Total time: 6.394 secs
   ```

2. 进入项目目录下的`build`文件夹，在`paoding-rose-base/build/libs`下会发现一个名为`ROOT.war`的东东，这货就是我们的web应用的war包

   ```
   ➜  paoding-rose-base git:(master) ls
   LICENSE               LICENSE.txt           README.md             build                   
   build.gradle          gradle.properties     paoding-rose-base.iml src
   ```

3. 复制`ROOT.war`到web容器中（如tomcat、resin、jetty...），然后启动，我的环境为`jetty port:8080`，启动容器后状态如下

   ```
   ➜  jetty-8080  tail -f logs/2014_11_06.stderrout.log
   十一月 07, 2014 1:19:15 上午 net.paoding.rose.RoseFilter initFilterBean
   信息: [init] exits from 'init/mappingTree'
   十一月 07, 2014 1:19:15 上午 net.paoding.rose.RoseFilter initFilterBean
   信息: [init] exits from 'init'
   十一月 07, 2014 1:19:15 上午 net.paoding.rose.RoseFilter printRoseInfos
   信息: [init] rose initialized, 2 modules loaded, cost 692ms!    (version=1.0.1-20100805)
   2014-11-07 01:19:15.632:INFO:ROOT:main: [init] rose initialized, 2 modules loaded, cost 692ms! (version=1.0.1-20100805)
   2014-11-07 01:19:15.660:INFO:oejsh.ContextHandler:main: Started o.e.j.w.WebAppContext@9d84476{/,file:/private/var/folders/lb/qlsp55px2yvb4mkwx3zxg42r0000gn/T/jetty-0.0.0.0-8080-ROOT.war-_-any-1291130023571217795.dir/webapp/,AVAILABLE}{/ROOT.war}
   2014-11-07 01:19:15.705:INFO:oejs.ServerConnector:main: Started ServerConnector@4fc5dce6{HTTP/1.1}{0.0.0.0:8080}
   2014-11-07 01:19:15.706:INFO:oejs.Server:main: Started @4384ms
   ```

4. 用`浏览器`或`curl`访问一记: `http://192.168.1.10:8080/hi`

   ```
   ➜  jetty-8080  curl http://192.168.1.10:8080/hi
   Hello World!%
   ```

#### 注意事项
