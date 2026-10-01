---
title: IDEA搭建Scala项目-helloworld
id: 81
date: 2024-09-27 19:10:53
auther: admin
excerpt: 安装Scala插件打开idea的settings进入plugins，搜索Scala安装安装Scala sdk官网https//www.scala-lang.org/download/scala2.html根据环境自由选择IDEA自动下载项目模块上右键，选择 add framework suppor
permalink: /?p=81
categories:
 - scala
tags: 
 - scala
 - idea
---

## 安装Scala插件

- 打开idea的settings
- 进入plugins，搜索Scala
- 安装

![](https://pic.feilewu.cn/uploads/2024/09/30/833f38d6-4e80-4a2e-90cb-b2983dd88fa6.png)


## 安装Scala sdk

### 官网

https://www.scala-lang.org/download/scala2.html

根据环境自由选择


![](https://pic.feilewu.cn/uploads/2024/09/30/5bab8a4f-0075-4788-9aca-df28a4a801d0.png)
### IDEA自动下载

![](https://pic.feilewu.cn/uploads/2024/09/30/2cd9c3d8-1be8-403a-96d3-8f93c3a69879.png)
- 项目模块上右键，选择 add framework support，选择Scala，然后下载适当的SDK

![](https://pic.feilewu.cn/uploads/2024/09/30/a7256b53-3c98-4442-ad04-dbaabb3f93f7.png)
## Hello word

### 新建Scala源码目录，新建Scala class，类型选Object

![](https://pic.feilewu.cn/uploads/2024/09/30/84f99319-e504-44a7-a6c9-d992bcff632b.png)
### 编写代码

```scala
object Demo {

  def main(args: Array[String]): Unit = {
    println("hello world")
  }

}
```

```shell
hello world
```

