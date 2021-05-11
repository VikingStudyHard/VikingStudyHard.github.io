---
title: 【Mac】在PyCharm中选择Anaconda中已有的环境
date: 2019.5.10
categories: 环境配置
tags: 
    - Anaconda
    - Mac
    - Pycharm
description: 在Pycharm中的多个项目共用一个位于Anaconda的python环境
---


# 已创建的项目更换成 Conda 环境
Preferences -> Project:xxxx -> Project Interpreter -> 右侧齿轮 -> Show all -> 下方加号
选择 Conda 已有环境，图中是NER：
{% img https://vikingstudyhard.github.io/images/Mac_anaconda/FC81546C495E398A81A147BC15262899.jpg 898 %}
# 添加 Anaconda 的环境变量
在`.zshrc`中添加`export PATH="/Applications/anaconda3/bin:$PATH"`，source激活该文件。
此时输入`conda --version`能得到版本号即设置成功。
# 其他Anaconda操作
`conda info --envs`或者`conda env list`列出所有的环境
`source activate NER`选择一个已有的环境，或者新建一个环境 `conda create -n env_name`
`conda install -n env_name xxx` conda在指定环境下安装包；`pip install xxxx`安装是在任何环境中安装python包。
`conda list`查看已安装的包
`conda remove`卸载指定包
`conda deactivate`推出虚拟环境

但是用[之前安装pyltp的方法](https://vikingstudyhard.github.io/2019/03/11/Mac_pyltp/)并不能成功，加到external libraries就自己没了，而且安装的macos 10.7对应的pyltp。
然后`pip install pyltp`就好了。。。
另外，安装Levenshtein用`pip install python-Levenshtein`