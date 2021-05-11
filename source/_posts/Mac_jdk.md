---
title: 【Mac】配置IntelliJ IDEA的JDK环境
date: 2018.12.10
categories: 环境配置
tags: 
    - JDK
    - Mac
    - IntelliJ IDEA
description: Mac OSX 默认的JDK版本过高，需要配置成JDK1.7
---

Mac OS X 默认的JDK 版本好高啊 在这里：
`/Library/Java/JavaVirtualMachines/jdk-9.0.1.jdk/Contents/Home`
而 Maven3.5.2 要对应1.7版本的jdk，所以要下一个低版本的jdk。
# 安装配置JDK
## 下载JDK
访问Oracle官网 
[http://www.oracle.com/](http://www.oracle.com/)
[https://www.oracle.com/technetwork/java/javase/downloads/index.html](https://www.oracle.com/technetwork/java/javase/downloads/index.html)

要下载1.7，登录 Oracle下载

https://www.oracle.com/technetwork/java/javase/downloads/index.html

得到 jdk-7u80-macosx-x64.dmg
## 安装
按操作来，默认安装在 `/Library/Java/JavaVirtualMachines/jdk1.7.0_80.jdk/`
## 配置JDK环境变量
### 临时有效（重启后失效）
1.修改家目录下的 `.bash_profile` 文件，插入以下两行：
```
export JAVA_HOME=/Library/Java/JavaVirtualMachines/jdk1.7.0_80.jdk/Contents/Home
export PATH=$JAVA_HOME/bin:$PATH
```
2.应用配置，并测试
`source ~/.bash_profile`
{% img https://vikingstudyhard.github.io/images/Mac_jdk/CEB683E12A1AAD8C1047D9BAE4197887.jpg  %}

现在：

{% img https://vikingstudyhard.github.io/images/Mac_jdk/595790964EDCCC89D924A097EA4C23BF.jpg  %}
### 永久有效
1.修改/etc目录下的 `profile` 文件，插入以下两行：
```
export JAVA_HOME=/Library/Java/JavaVirtualMachines/jdk1.7.0_80.jdk/Contents/Home
export PATH=$JAVA_HOME/bin:$PATH
```
2.使修改的文件生效：`source /etc/profile`

## 查看已安装的jdk版本及其安装目录
`/usr/libexec/java_home -V`

{% img https://vikingstudyhard.github.io/images/Mac_jdk/F86F35C0548EFE104A5DEFCC6DC5859D.jpg  %}


# 在IntelliJ IDEA配置相应的jdk
打开 IntelliJ IDEA -> file -> ProjectStructure
