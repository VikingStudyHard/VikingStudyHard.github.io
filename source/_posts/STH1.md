---
title: 【软件测试】01 曾经遇到的错误
date: 2018-03-07 
categories: [软件测试, 本科课程]
tags: 
    - SoftwareTest
description: Briefly describe an error from my past projects that I have completed
---

在数据库大作业中需要完成一个学生选课及成绩管理系统，实验工具为：Eclipse（swt）、Navicat

### 错误内容以及发现的过程
在系统中经过多次增删改查的操作后，出现闪退的现象，报错“too many connections”
### 错误出现的原因
对于每次建立的链接没有及时进行关闭，即每次
```java
DB db = new db();
```
之后写完执行语句就收尾了，都没有进行
```java
db.close();
```
由于mysql默认的连接只有100个，所以大量的connect之后，就会出现Too many connections的错误。

### 错误带来的影响
系统突然出现闪退的现象，需要重新运行。但是在实际中这就会导致严重的问题与漏洞。
### 经验教训

最稳妥的方法就是及时释放一些没用的连接。
如果必要，可以通过修改mysql配置文件来加大允许连接的数量。