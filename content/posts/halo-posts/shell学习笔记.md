---
title: shell学习笔记
id: 79
date: 2024-09-27 19:10:53
auther: admin
cover: 
excerpt: 
permalink: /?p=79
categories:
 - 脚本
tags: 
 - shell
---

## 变量

### 使用和创建变量

```shell
your_name="qinjx"
echo $your_name
echo ${your_name}
```

### 删除变量

```shell
#!/bin/sh
myStr="deleted string"
unset myStr
echo $myStr
```

## 字符串

- 单引号和双引号的区别

  - 单引号内的字符串会原样输出
  - 双引号中的变量会进行替换

- 计算字符串长度

  ```shell
  string="abcd"
  echo ${#string} #输出 4
  ```

- 截断字符串

  ```shell
  string="runoob is a great site"
  echo ${string:1:4} # 输出 unoo
  ```

## 数组

```shell
数组名=(值1 值2 ... 值n)
```

```shell
#!/bin/bash
array=(1 2 3 4)
echo ${array[1]}
```

### 获取数组长度

```shell
echo ${array_name[@]}
# 或者
length=${#array_name[*]}
```

### 获取数组单个元素的长度

```shell
# 取得数组单个元素的长度
lengthn=${#array_name[n]}
```

## 注释

### 单行

```shell
# 注释内容
```

### 多行

```shell
:<<EOF
注释内容...
注释内容...
注释内容...
EOF
```

```shell
:<<'
注释内容...
注释内容...
注释内容...
'
```

```shell
:<<!
注释内容...
注释内容...
注释内容...
!
```

## 参数接收

```shell
#!/bin/bash
echo "Shell 传递参数实例！";
echo "执行的文件名：$0";
echo "第一个参数为：$1";
echo "第二个参数为：$2";
echo "第三个参数为：$3";
```

```java
./demo.sh 1 2 3
```

| 参数处理 | 说明                                                         |
| :------- | :----------------------------------------------------------- |
| $#       | 传递到脚本的参数个数                                         |
| $*       | 以一个单字符串显示所有向脚本传递的参数。 如"$*"用「"」括起来的情况、以"$1 $2 … $n"的形式输出所有参数。 |
| $$       | 脚本运行的当前进程ID号                                       |
| $!       | 后台运行的最后一个进程的ID号                                 |
| $@       | 与$*相同，但是使用时加引号，并在引号中返回每个参数。 如"$@"用「"」括起来的情况、以"$1" "$2" … "$n" 的形式输出所有参数。 |
| $-       | 显示Shell使用的当前选项，与[set命令](https://www.runoob.com/linux/linux-comm-set.html)功能相同。 |
| $?       | 显示最后命令的退出状态。0表示没有错误，其他任何值表明有错误。 |

## 运算符

### 算数运算符

| 运算符 | 说明                                          | 举例                          |
| :----- | :-------------------------------------------- | :---------------------------- |
| +      | 加法                                          | `expr $a + $b` 结果为 30。    |
| -      | 减法                                          | `expr $a - $b` 结果为 -10。   |
| *      | 乘法                                          | `expr $a \* $b` 结果为  200。 |
| /      | 除法                                          | `expr $b / $a` 结果为 2。     |
| %      | 取余                                          | `expr $b % $a` 结果为 0。     |
| =      | 赋值                                          | a=$b 将把变量 b 的值赋给 a。  |
| ==     | 相等。用于比较两个数字，相同则返回 true。     | [ $a == $b ] 返回 false。     |
| !=     | 不相等。用于比较两个数字，不相同则返回 true。 | [ $a != $b ] 返回 true。      |

### 关系运算符

| 运算符 | 说明                                                  | 举例                       |
| :----- | :---------------------------------------------------- | :------------------------- |
| -eq    | 检测两个数是否相等，相等返回 true。                   | [ $a -eq $b ] 返回 false。 |
| -ne    | 检测两个数是否不相等，不相等返回 true。               | [ $a -ne $b ] 返回 true。  |
| -gt    | 检测左边的数是否大于右边的，如果是，则返回 true。     | [ $a -gt $b ] 返回 false。 |
| -lt    | 检测左边的数是否小于右边的，如果是，则返回 true。     | [ $a -lt $b ] 返回 true。  |
| -ge    | 检测左边的数是否大于等于右边的，如果是，则返回 true。 | [ $a -ge $b ] 返回 false。 |
| -le    | 检测左边的数是否小于等于右边的，如果是，则返回 true。 | [ $a -le $b ] 返回 true。  |

...

## 流程控制

### if else

```shell
if condition
then
    command1 
    command2
    ...
    commandN 
fi
```

```shell
if [ $(ps -ef | grep -c "ssh") -gt 1 ]; then echo "true"; fi
```

```shell
if condition1
then
    command1
elif condition2 
then 
    command2
else
    commandN
fi
```

例子

```shell
a=10
b=20
if [ $a == $b ]
then
   echo "a 等于 b"
elif [ $a -gt $b ]
then
   echo "a 大于 b"
elif [ $a -lt $b ]
then
   echo "a 小于 b"
else
   echo "没有符合的条件"
fi
```

### for 循环

```shell
for var in item1 item2 ... itemN
do
    command1
    command2
    ...
    commandN
done
```

```shell
for var in item1 item2 ... itemN; do command1; command2… done;
```

### while 语句

```shell
while condition
do
    command
done
```

```shell
#!/bin/bash
int=1
while(( $int<=5 ))
do
    echo $int
    let "int++"
done
```

### until 循环

```shell
#!/bin/bash
a=0
until [ ! $a -lt 10 ]
do
   echo $a
   a=`expr $a + 1`
done
```

### case ... esac

```shell
echo '输入 1 到 4 之间的数字:'
echo '你输入的数字为:'
read aNum
case $aNum in
    1)  echo '你选择了 1'
    ;;
    2)  echo '你选择了 2'
    ;;
    3)  echo '你选择了 3'
    ;;
    4)  echo '你选择了 4'
    ;;
    *)  echo '你没有输入 1 到 4 之间的数字'
    ;;
esac
```

### break和continue

## 函数

```shell
[ function ] funname [()]

{

    action;

    [return int;]

}
```

