---
title: 卷积神经网络
date: 2019.7.12
categories:  Machine learning

description: CNN入门
mathjax: true
---

卷积：加权求和
# 三大思想
**局部感知** 滑动窗口
**权值共享** 共同使用一个卷积核
**池化**
# 卷积层
{% img https://vikingstudyhard.github.io/images/MachineLearning_CNN/C5D7501E6165090E30D6E497BE27CD58.png 500 %}

假如我们有一张$6\times 6$的灰度图片，再定义一个$3\times 3$的矩阵，称为卷积核(kernal)、过滤器(filter)。上述过滤器就是用来识别垂直边缘。

通过这个过滤器给一张图片查找垂直边缘和水平边缘的效果如下：
{% img https://vikingstudyhard.github.io/images/MachineLearning_CNN/B6508F6626C4DC578BBE072B10A404F0.png 500 %}

卷积运算：3*1+1*1+2*1+0*0+5*0+7*0+1*-1+8*-1+2*-1=-5 加权求和


使用多个过滤器来识别不同的浅层特征：
{% img https://vikingstudyhard.github.io/images/MachineLearning_CNN/165DA66662D79CEEFB6F87B5E49D8C20.png 500 %}
上图通过一个垂直边缘过滤器和水平边缘过滤器来让神经网络同时识别垂直边缘和水平边缘。
每个过滤器是$3\times 3 \times 3$的(因为RGB有3个颜色通道，所以最后还要乘以3)，所以过滤器有27个值。步长为1，输出两个$4\times 4$的矩阵，最后把它们重叠在一起后输入给下一层。 

# padding 边界扩充
- 卷积运算后，输出图片尺寸缩小
- 越是边缘的像素点，对于输出的影响越小，因为卷积运算在移动的时候到边缘就结束了。中间的像素点有可能会参与多次计算，但是边缘像素点可能只参与一次。所以我们的结果可能会丢失边缘信息。

{% img https://vikingstudyhard.github.io/images/MachineLearning_CNN/068E044F5EA99FCFA85F6995060FACEB.png 284 %}
在图片外围补充一些像素点，把这些像素点初始化为0。

经过padding之后，$n\times n$的图片尺寸变为 $(n+2p) \times (n+2p)$，filter 尺寸为 $f\times f$，步长为1，则卷积后的图片尺寸为 $(n+2p-f+1) \times (n+2p-f+1)$。若要保证卷积前后图片尺寸不变，则p应满足：$ p = \frac{f-1}{2}$ 

# Stride 步长
Stride表示filter在原图片中水平方向和垂直方向每次的步进长度。
【吴恩达】任意一个卷积层输出图片的大小 $n'=\left \lfloor  \frac{(n-f+2p)}{S}+1 \right \rfloor$。过滤器必须完全处于图像中或者填充之后的图像区域内才输出相应结果。过滤器移动到了外面，就忽略掉（忽略的通常是padding）

# ReLU
每次的卷积操作后都使用了一个叫做ReLU的操作。ReLU 表示修正线性单元（Rectified Linear Unit），是一个非线性操作。
定义：$output = max(input,0)$。目的：使数据**非线性化**。


# Pooling 池化层
作用：降低维数，以减少网络中的参数和计算次数。缩短训练时间并控制过度拟合。
最常见的池类型是max pooling（最大值池化），它在每个窗口中占用最大值。需要事先指定这些窗口大小。这会降低特征图的大小，同时保留重要信息。
还有mean-pooling，邻域内特征点只求平均。
# 全连接层
目的：卷积和池化层的输出高级特征，全连接层使用高级特征把输入图像基于训练数据集进行分类。

#总览
{% img https://vikingstudyhard.github.io/images/MachineLearning_CNN/594D5FBFC524B9F6CD9033CB9F337F48.jpg 582 %}
从左往右依次是输入层，卷积层，池化层，输出层
池化层到输出层是全连接的。

# Reference
https://testerhome.com/topics/11891
https://testerhome.com/topics/12383
https://mp.weixin.qq.com/s/bz06wv6yWQbQYMLfVdcTTg
https://www.jianshu.com/p/df12b95dedee