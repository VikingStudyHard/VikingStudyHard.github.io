---
title: 【Mac】用Anaconda配置pyTorch的环境
date: 2019.7.10
categories: 环境配置
tags: 
    - Mac
    - Anaconda
    - pyTorch
description: 搭建NLP环境，用Anaconda配置pyTorch
---
# 配置虚拟环境
1.创建一个名叫pytorch36的虚拟的环境
`conda create -n pytorch36 python=3.6`
2.激活环境
`source activate pytorch36`
3.安装pytorch
`conda install pytorch torchvision -c pytorch`

4.验证安装是否成功：
```
import torch
print(torch.rand(2, 3))
```
执行后输出类似于：
```
tensor([[0.6122, 0.4238, 0.0422],
        [0.8809, 0.5974, 0.5142]])
```

{% note warning %} 
如果在终端出现了报错，可能是`~/.zshrc`的问题。
{% endnote %}

{% img https://vikingstudyhard.github.io/images/Mac_pytorch/29EF6A3CC431799C8CD85A417A15BDF4.jpg 504 %}
结果并没有成功，python还是default的版本。
在`~/.zshrc`中，有这句话，注释掉，并source此文件：
{% img https://vikingstudyhard.github.io/images/Mac_pytorch/4D0FE868764ABE58C7014D283DA9158B.jpg 595 %}
重新激活pytorch36环境，重复上述过程，可见python已经是当前环境的版本：
{% img https://vikingstudyhard.github.io/images/Mac_pytorch/29A2BBDD128C1B7A4FF955C384379C23.jpg 541 %}

{% note warning %} 
直接在pycharm中引用这个已存在的环境的话，就没有这个问题。
{% endnote %}

5.退出虚拟环境
`conda deactivate`
# 参考
官网 https://pytorch.org/get-started/locally/
参考 https://blog.csdn.net/ccbrid/article/details/80418769