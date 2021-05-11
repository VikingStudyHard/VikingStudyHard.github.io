---

title: 【Mac】安装 Pyltp

date: 2019.3.11

categories: 环境配置

tags: 
 - Mac
 - Python
 - NLP
 - Pycharm

description: 本文提供了在MacOS 10.13上相对简便没有坑的Pyltp的安装方法

---
在MacOS 10.13上安装pyltp，pip的方法报错，[从源代码编译安装](https://blog.csdn.net/huacha__/article/details/83721399)到了install的步骤时也出现`command 'clang++' failed with exit status 1`的报错。
本文提供了在MacOS 10.13上相对简便没有坑的Pyltp的安装方法。

# pyltp介绍
pyltp 是 LTP 的 Python 封装，提供了分词，词性标注，命名实体识别，依存句法分析，语义角色标注的功能。

github网址： https://github.com/HIT-SCIR/pyltp
在线文档： https://pyltp.readthedocs.io/zh_CN/latest/api.html

# 操作环境

操作平台：Mac OS 10.13
python版本：python 3.6.3

# Mac 10.13 下安装 Pyltp

1.打开[https://pypi.org/project/pyltp/\#files](https://pypi.org/project/pyltp/#files)，下载[pyltp-0.2.1.tar.gz ](https://files.pythonhosted.org/packages/aa/72/2d88c54618cf4d8916832950374a6f265e12289fa9870aeb340800a28a62/pyltp-0.2.1.tar.gz)。
解压后文件结构为：

{% img https://vikingstudyhard.github.io/images/Mac_pyltp/4C5544312BE5791D791C2B7C93562104.jpg 207 %}

2.修改setup.py的第121行
```python setup.py 
if not 'MACOSX_DEPLOYMENT_TARGET' in os.environ:
    os.environ['MACOSX_DEPLOYMENT_TARGET'] = '10.12'
```

将 10.12 改成 10.13

3.开始安装
```
sudo python setup.py install
```
出现类似于下图所示说明，即为安装成功。

{% img https://vikingstudyhard.github.io/images/Mac_pyltp/5470DC78C00B829F68218D9D2CA61002.jpg 1700 %}

因为该pyltp-0.2.1.tar.gz的ltp文件夹是完整的，所以不用再单独下载。

4.添加到 external libraries 里，最终的文件结构如下图所示：

{% img https://vikingstudyhard.github.io/images/Mac_pyltp/B60396087CD20B9ECCCBA5F550086D6D.jpg 475 %}


5.下载模型文件 ltp_data_v3.4.0.zip

下载地址： http://ltp.ai/download.html

{% img https://vikingstudyhard.github.io/images/Mac_pyltp/746EFCF95C23CAF02082DB703CAC3155.jpg 238 %}


设置ltp模型目录的路径，就能用了。
```
LTP_DATA_DIR = './ltp_data_v3.4.0'
```
# 检查安装是否成功
可以在python中输入`from pyltp import SentenceSplitter`看看有没有报错


# 依存句法分析
使用 pyltp 进行依存句法分析示例如下
```python
# -*- coding: utf-8 -*-
import os
LTP_DATA_DIR = '/path/to/your/ltp_data'  # ltp模型目录的路径
par_model_path = os.path.join(LTP_DATA_DIR, 'parser.model')  # 依存句法分析模型路径，模型名称为`parser.model`

from pyltp import Parser
parser = Parser() # 初始化实例
parser.load(par_model_path)  # 加载模型

words = ['元芳', '你', '怎么', '看']
postags = ['nh', 'r', 'r', 'v']
arcs = parser.parse(words, postags)  # 句法分析

print "\t".join("%d:%s" % (arc.head, arc.relation) for arc in arcs)
parser.release()  # 释放模型
```
其中，words 和 postags 分别为分词和词性标注的结果。同样支持Python原生的list类型。

结果如下
```python
4:SBV       4:SBV   4:ADV   0:HED
```
`arc.head` 表示依存弧的父节点词的索引。ROOT节点的索引是0，第一个词开始的索引依次为1、2、3…

`arc.relation` 表示依存弧的关系。

`arc.head` 表示依存弧的父节点词的索引，`arc.relation` 表示依存弧的关系。

标注集请参考[依存句法关系](https://ltp.readthedocs.io/zh_CN/latest/appendix.html#id5)。

# 参考
https://blog.csdn.net/huacha__/article/details/83721399
https://pyltp.readthedocs.io/zh_CN/latest/api.html