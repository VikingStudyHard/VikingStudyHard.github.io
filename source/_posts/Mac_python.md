---
title: 【Mac】配置Python的自然语言处理环境
date: 2018-01-30
categories: 环境配置
tags: 
    - Python
    - Mac
    - NLP
    - Pycharm
    
description: Mac配置Python的自然语言处理环境
---


1.下载python

<https://www.python.org/>

2.Mac OS上安装setuptools

可以参考 

<https://pypi.python.org/pypi/setuptools>

```
curl https://bootstrap.pypa.io/ez_setup.py -o - | python
```

3.安装pip 

在终端下运行
```
sudo easy_install pip
```

4.安装 Numpy 和 matplotlib
运行
```
sudo pip install -U numpy matplotlib
```

5.安装 pyyaml 和 nltk 
运行
```
sudo pip install -U pyyaml nltk
```