---
title: 【Mac】通过Anaconda搭建不同Python版本的Tensorflow环境
date: 2018.11.12
categories: 环境配置
tags: 
    - Python
    - Tensorflow
    - Mac
    - Anaconda
    - Pycharm
    
description: 通过Anaconda搭建Python2和Python3的Tensorflow环境
---
从 1.2 版本开始，在 Mac OS X 上 TensorFlow 不再支持 GPU。
# Mac + Python3 + Tensorflow
##安装 python3
`brew install python3`
##安装TensorFlow
### 安装pip
`sudo curl https://bootstrap.pypa.io/ez_setup.py -o - | sudo python`
`sudo easy_install pip`
`sudo easy_install --upgrade pip` 更新pip
其中
pip, for Python 2.7
pip3, for Python 3.n.
{% img https://vikingstudyhard.github.io/images/Mac_tensorflow/7909BE63F31D6064D57D3C6874A63CDB.jpg %}

### 用pip3下载tensorflow 3的版本

直接安装 `pip3 install tensorflow` 会提示一个当先版本匹配不到的错误提示：
```
pip3 install tensorflow 
Collecting tensorflow
Could not find a version that satisfies the requirement tensorflow (from versions: ) No matching distribution found for tensorflow
```

所以要用以下命令
安装1.3.0版本：
`pip3 --default-timeout=10000 install --upgrade https://storage.googleapis.com/tensorflow/mac/cpu/tensorflow-1.3.0-py3-none-any.whl`

安装1.8.0版本：
`pip3 --default-timeout=10000 install --upgrade https://storage.googleapis.com/tensorflow/mac/cpu/tensorflow-1.8.0-py3-none-any.whl`

## Anaconda的下载
[https://www.anaconda.com/download/#macos](https://www.anaconda.com/download/#macos)

## 使用 Anaconda 创建 Python3 的虚拟环境
[https://www.tensorflow.org/install/install_mac#installing_with_anaconda](https://www.tensorflow.org/install/install_mac#installing_with_anaconda)
1.执行以下命令创建名为 python3 的 conda 环境:
`$ conda create -n python3`
2.执行以下命令激活 conda 环境：
```
$ source activate python3
 (python3)$  # Your prompt should change
```

3.验证安装

运行一个小的 TensorFlow 程序
在一个 shell 中执行 Python：
`$ python`
在 python 交互式 shell 中输入以下小程序：

```
import tensorflow as tf
hello = tf.constant('Hello, TensorFlow!')
sess = tf.Session()
print(sess.run(hello))
```
如果系统输出以下内容，则安装成功：
`Hello, TensorFlow!`

{% img https://vikingstudyhard.github.io/images/Mac_tensorflow/6138282E7ABA5F256B6511637CBE632E.jpg %}
输出结果中的b是bytes的意思，python3的语法。加‘b’表示按位字符串，‘r’原始意义（常用在带有转义字符的路径中，表示不转义）; ‘u’加字符串表示unicode编码字符串，同时适用于中英文。

4.停用环境
`source deactivate`
5.删除环境
`conda remove -n python3`
6.如果要卸载 TensorFlow，执行下面的命令
`$ pip3 uninstall tensorflow `

# Mac + Python2.7 + Tensorflow1.8.0
## 安装对应 Python2 的 Tenserflow
`pip --default-timeout=10000 install --upgrade  https://download.tensorflow.google.cn/mac/cpu/tensorflow-1.8.0-py2-none-any.whl`
## 使用 Anaconda 创建 Python2 的虚拟环境
1.创建一个名叫python27的虚拟的环境 `conda create -n python27 python=2.7`
2.激活环境 `source activate python27`
可以看到：
{% img https://vikingstudyhard.github.io/images/Mac_tensorflow/257BEDC068E5DA96150FE99FD97D431A.jpg %}
3.退出环境 `source deactivate`
4.删除环境 `conda remove -n python27 --all`