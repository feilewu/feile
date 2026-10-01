---
title: SpringBoot配置文件——加载顺序
id: 80
date: 2024-09-27 19:10:53
auther: admin
cover: 
excerpt: SpringBoot配置文件——加载顺序配置文件所在目录file./config/   根目录中的config目录中file./    根目录中classpath/config/   resource目录中的config目录下classpath/   resource目录下（类路径下 cla
permalink: /?p=80
categories:
 - springboot
tags: 
 - java
---

# SpringBoot配置文件——加载顺序

### 配置文件所在目录

![](https://pic.feilewu.cn/uploads/2024/09/30/a9298a51-96ba-496a-9dbf-57337e30a218.png)
- file:./config/   根目录中的config目录中
- file:./    根目录中
- classpath:/config/   resource目录中的config目录下
- classpath:/   resource目录下（类路径下 classpath）

### 读取顺序（优先级顺序）

如果在不同的目录中存在多个配置文件，它的读取顺序是：
 1、config/application.properties（项目根目录中config目录下）
 2、config/application.yml
 3、application.properties（项目根目录下）
 4、application.yml
 5、resources/config/application.properties（项目resources目录中config目录下）
 6、resources/config/application.yml
 7、resources/application.properties（项目的resources目录下）
 8、resources/application.yml

结论：

配置的优先级如上所示，如果同一个配置配置在多个配置文件中，则优先使用先读取到的，也就是说，1-8的优先级是由高到低的

PS：

创建SpringBoot项目时，一般的配置文件放置在项目的resources目录下，因为配置文件的修改，通过热部署不用重新启动项目，而热部署的作用范围是classpath下



参考：

springboot官方文档：https://docs.spring.io/spring-boot/docs/2.3.3.RELEASE/reference/html/spring-boot-features.html#boot-features-external-config

