---
title: "无法将交换文件 /vmfs/volumes/.../XXXXX.vswp 从 0 KB 扩展到 6291456 KB : No space left on device"
id: 89
date: 2024-09-27 19:10:53
auther: admin
cover: 
excerpt: 
permalink: /?p=89
categories:
tags: 
 - esxi
 - 虚拟机
---

在编辑虚拟机，增加内存时，出现如下的错误


![](https://pic.feilewu.cn/uploads/2024/09/30/5334e50b-0f1d-4011-a9d8-4aa4ae865253.png)
解决方案：

在编辑虚拟机，增加内存时，将预留内存也一并分配

![](https://pic.feilewu.cn/uploads/2024/09/30/35e46a35-213b-4853-86e4-4022c399d675.png)