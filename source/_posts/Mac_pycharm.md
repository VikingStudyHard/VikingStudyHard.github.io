---
title: 【Mac】我的Pycharm小技巧
date: 2019.5.31
categories: 环境配置
tags: 
    - Pycharm
    - Mac
description: Pycharm小技巧，更新ing
---
# 执行路径未更新
## 问题
Pycharm 文件更改路径后，执行时出现了路径找不到的错误。
打印工作路径 print(os.getced())
## 解决
MACOS快捷键 `Command + Alt + r`，选择Edit Configurations。选择该文件，找到 Working directory，在此处修改成正确的路径，点击 Apply 即可。
> https://blog.csdn.net/qq_24076135/article/details/78028822

# Find in path
超级有用的全局查找功能。
MACOS快捷键为`Command + Shift + f`或者菜单：Edit --> Find --> Find in Path

# 修改文件默认打开方式
## 问题
新建时不小心设置为用“Text”打开，想要修改文件默认打开方式。
## 解决
Editor —> File Types，在text中移除该文件。

>https://blog.csdn.net/qq_25046261/article/details/81666118

# 设置External Libraries
菜单 Preference --> Project Interpreter 输入框后面的齿轮，选择Show All。
在下面有一排按钮，点击最后一个“Show paths for selected interpreter”，在弹出页面点击加号。
> https://blog.csdn.net/hfutdog/article/details/81711531