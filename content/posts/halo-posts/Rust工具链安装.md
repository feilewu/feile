---
title: Rust工具链安装
id: 83680867-d20a-472c-bed2-070631a717cf
date: 2024-10-01 13:46:53
auther: admin
cover: 
excerpt: Windows 安装MSYS2 https//www.msys2.org/ 安装工具链 pacman -S mingw-w64-ucrt-x86_64-gcc 查看gcc版本 gcc --version 安装Rust 配置windows环境变量 将下面述地址配置到w
permalink: /?p=83680867-d20a-472c-bed2-070631a717cf
categories:
tags: 
 - rust
---


# Windows

## 安装MSYS2

https://www.msys2.org/
![](https://pic.feilewu.cn/uploads/2024/10/01/3d4baffe-c22f-41ce-9f1b-44227a2332a1.png)

安装工具链
```shell
pacman -S mingw-w64-ucrt-x86_64-gcc
```

查看gcc版本

```shell
gcc --version
```

## 安装Rust


### 配置windows环境变量

将下面述地址配置到windows系统环境变量

```
C:\msys64\ucrt64\bin
```


### 执行安装Rust

MSYS2终端内执行

```shell
curl https://sh.rustup.rs -sSf | sh
```


一些选择
```
Current installation options:


   default host triple: x86_64-pc-windows-msvc
     default toolchain: stable (default)
               profile: default
  modify PATH variable: yes

1) Proceed with standard installation (default - just press enter)
2) Customize installation
3) Cancel installation
>2

I'm going to ask you the value of each of these installation options.
You may simply press the Enter key to leave unchanged.

Default host triple? [x86_64-pc-windows-msvc]

# 手动填写
x86_64-pc-windows-gnu


Default toolchain? (stable/beta/nightly/none) [stable]


Profile (which tools and data to install)? (minimal/default/complete) [default]


Modify PATH variable? (Y/n)
y


Current installation options:


   default host triple: x86_64-pc-windows-gnu
     default toolchain: stable
               profile: default
  modify PATH variable: yes

1) Proceed with selected options (default - just press enter)
2) Customize installation
3) Cancel installation
>1

```

### 检验安装结果

windows终端执行

```
rustup update

rustc -V

cargo -V
```


