---
title: Scala基础--关键字，数据类型，函数
id: 82
date: 2024-09-27 19:10:53
auther: admin
cover: 
excerpt: 
permalink: /?p=82
categories:
 - scala
tags: 
 - scala
---



## 基础语法

### 注意点

- Scala 区分大小写
- 类名类似于Java
- 方法名命名类似于Java

老规矩，先上Hello World

```scala
object Demo {

  def main(args: Array[String]): Unit = {
    println("hello world")
  }

}
```

### 关键字

| abstract  | case     | catch    | class   |
| --------- | -------- | -------- | ------- |
| def       | do       | else     | extends |
| false     | final    | finally  | for     |
| forSome   | if       | implicit | import  |
| lazy      | match    | new      | null    |
| object    | override | package  | private |
| protected | return   | sealed   | super   |
| this      | throw    | trait    | try     |
| true      | type     | val      | var     |
| while     | with     | yield    |         |
| -         | :        | =        | =>      |
| <-        | <:       | <%       | >:      |
| #         | @        |          |         |

### 注释

注释也和Java类似

```scala
object Demo {
  
  /**
   * 注释
   * @param args
   */
  def main(args: Array[String]): Unit = {
    //注释
    println("hello world")
  }

}
```

### 导包

```scala
import scala.xml._
```

与Java不同的是，Scala导入包内的所有类用的是下划线而不是星号

```scala
import scala.collection.mutable.HashMap
```

导入类

```scala
import scala.collection.immutable.{TreeMap, TreeSet}
```

导入包中的几个类，用花括号括起来，而不必像Java中一样得写N行



## 数据类型

| 数据类型 | 描述                                                       |
| -------- | ---------------------------------------------------------- |
| Byte     | 8位有符号值。范围从-128到127                               |
| Short    | 16位有符号值。范围从-32768至32767                          |
| Int      | 32 位有符号值。范围从 -2147483648 to 2147483647            |
| Long     | 64位有符号值。 从-9223372036854775808到9223372036854775807 |
| Float    | 32位IEEE754单精度浮点数                                    |
| Double   | 64位IEEE754双精度浮点数                                    |
| Char     | 16位无符号Unicode字符。范围由U+0000至U+FFFF                |
| String   | 字符序列                                                   |
| Boolean  | 无论是字面true或false字面                                  |
| Unit     | 对应于没有值                                               |
| Null     | 空或空引用                                                 |
| Nothing  | 每一个其他类型的子类型; 包括无值                           |
| Any      | Any类型的超类型;任何对象是任何类型                         |
| AnyRef   | 任何引用类型的超类型                                       |

```scala
def base(): Unit ={
    val a:Int = 1
    val b:Short = 2
    val c:Long = 2L
    val d:Float = 0.02F
    val f:Double = 0.223D
    val e:Char = '2'
    val g:String = "hello"
    //....
  }
```

## 变量

关键词var和val，var表示可变变量，val表示常量（一旦初始化，其值不可更改）

```scala
var lily:String = "Lily"
lily = "ss"
val bob:String = "Bob"
bob = "ss"//会便宜报错
```

## 访问修饰符

Scala 访问修饰符有private 、protected 

```scala
```

## 函数

```scala
object Test {
   def main(args: Array[String]) {
        println( "Returned Value : " + addInt(5,7) );
   }
    
   //函数，类似于Java方法
   //高阶用法后面再说
   def addInt( a:Int, b:Int ) : Int = {
      var sum:Int = 0
      sum = a + b
      return sum
   }
}
```

### Scala允许参数以非定义顺序传递

```scala
object Demo {
    
    def printS(a:Int,b:Int): Unit = {
        println(a)
        println(b)
  	}
    
    def main(args: Array[String]) {
        printS(b=3,a=4)
   }
    
}
```

结果

```shell
4
3
```

### 可变参数传参

```scala
object Test {
   def main(args: Array[String]) {
        printStrings("Hello", "Scala", "Python");
   }
   def printStrings( args:String* ) = {
      var i : Int = 0;
      for( arg <- args ){
         println("Arg value[" + i + "] = " + arg );
         i = i + 1;
      }
   }
}
```

结果

```shell
Arg value[0] = Hello
Arg value[1] = Scala
Arg value[2] = Python
```

###  默认参数值

这点和python差不多

```scala
def addInt( a:Int=5, b:Int=7 ) : Int = {
    a+b
}
```

调用的时候

- 可以不传入参数
- 传入的参数从左开始赋值，如果add(3)，则输出值10

