---
title: centos7编译安装python3.11
id: 3f5ffb9b-c7a8-4b1d-a71d-cd329d9ff405
date: 2024-10-06 22:00:46
auther: admin
cover: 
excerpt: sudo yum install gcc openssl-devel bzip2-devel libffi-devel wget wget https//www.python.org/ftp/python/3.11.4/Python-3.11.4.tgztar -xvf Python-3.1
permalink: /?p=3f5ffb9b-c7a8-4b1d-a71d-cd329d9ff405
categories:
tags: 
 - centos7
---


```
sudo yum install gcc openssl-devel bzip2-devel libffi-devel wget
```

```
wget https://www.python.org/ftp/python/3.11.4/Python-3.11.4.tgz

tar -xvf Python-3.11.4.tgz
```


```
# 安装源码编译需要的编译环境 
yum -y install gcc zlib zlib-devel libffi libffi-devel readline-devel 

# 安装openssl11，后期的pip3安装网络相关模块需要用到ssl模块
yum install openssl-devel openssl11 openssl11-devel 

# 设置编译FLAG，以便使用最新的openssl库 
export CFLAGS=$(pkg-config --cflags openssl11) 
export LDFLAGS=$(pkg-config --libs openssl11)

# 进入刚解压缩的目录 
cd /home/hadoop/opt/Python-3.11.4 

#指定python3的安装目录为 /usr/python 并使用ssl模块，指定目录好处是后期删除此文件夹就可以完全删除软件了。 

./configure --prefix=/usr/local/python3.11.4/python3.11.4 --with-ssl # 源码编译并安装,时间会持续几分钟 

make && make install

```

```
sudo ln -s /usr/local/python3.11.4/bin/python3.11 /usr/bin/python3.11
```


```
# 以下可选

# 可以删除，也可备份，按需操作即可 
sudo rm -f /usr/bin/python 
sudo rm -f /usr/bin/pip

sudo ln -s /home/hadoop/opt/python3.11.4/bin/pip3 /usr/bin/pip3 
sudo ln -s /home/hadoop/opt/python3.11.4/bin/python3 /usr/bin/python 
sudo ln -s /home/hadoop/opt/python3.11.4/bin/pip3 /usr/bin/pip

```
