---
title: 【Andrew Ng】Machine Learning | 2 Linear regression with one variable

date: 2019.01.18

categories: Machine learning

tags: 
 - Andrew Ng

description: 第二章 Linear regression with one variable
mathjax: true

---
第二章 Linear regression with one variable 单变量线性回归
# 1 Model representation 模型描述
<div align=center>
  <img width=567 src="https://vikingstudyhard.github.io/images/MachineLearning_AndrewNg_2/B9767009C0EA72B2F9416B1B7FB0DACE.jpg" >
</div>

有监督学习：数据中给出了每个样本对应的正确答案
回归问题：预测具体数值输出

预测房价
## 训练集
<div align=center>
  <img width=583 src="https://vikingstudyhard.github.io/images/MachineLearning_AndrewNg_2/FF8DAC215FBEB505C1D890ED2F50DA8C.jpg" >
</div>

$m$ 训练样本的数量
$x$ 输入变量/特征
$y$ 输出变量/要预测的目标变量
$(x,y)$ 一组训练样本
$(x^{(i)},y^{(i)})$ 第i组训练样本，i是index不是幂。

## 假设函数 hypothesis function
<div align=center>
  <img width=406 src="https://vikingstudyhard.github.io/images/MachineLearning_AndrewNg_2/4071B2EB60BF7550D894FC50C1A0077F.jpg" >
</div>

$ h_\theta (x) = \theta_0 + \theta_1x $ 缩写为 $h(x)$
<div align=center>
  <img width=253 src="https://vikingstudyhard.github.io/images/MachineLearning_AndrewNg_2/146F4D691533804D5236D130D3BF9BDF.jpg" >
</div>

称为：
Linear regression with one variable 一元线性回归
Univariate linear regression 单变量线性回归


# 2 Cost function 代价函数
## 代价函数意义
代价函数用来衡量假设函数的准确性。

假设函数中的 $\theta_0$ $\theta_1$ 称为模型参数，如何选择参数？
<div align=center>
  <img width=314 src="https://vikingstudyhard.github.io/images/MachineLearning_AndrewNg_2/329D21C545ED8110919FEFD6EA892951.jpg" >
</div>

## 代价函数公式推导
最小化问题 minimize: 
使每组样本的 $\left ( h_\theta (x) - y \right )^2$ 最小，即使下式最小
$\sum_{i=1}^{m}\left ( h_\theta (x^{(i)}) - y^{(i)} \right )^2$ ，也就是预测值和实际值的差的平方最小。

平均误差：上式乘以$ \frac{1}{m}$。

通常求这个值的一半，也就是乘以$ \frac{1}{2m}$。平均值减半是为了便于计算梯度下降，因为平方函数的导数项将抵消$ \frac{1}{2}$项。

找到使上式最小的$\theta_0$ $\theta_1$：$\underset{\theta_0 ,\theta_1}{minimize}\frac{1}{2m}\sum_{i=1}^{m}\left ( h_\theta (x^{(i)}) - y^{(i)} \right )^2$

## 代价函数定义
$$J(\theta_0,\theta_1)=\frac{1}{2m}\sum_{i=1}^{m}\left ( h_\theta (x^{(i)}) - y^{(i)} \right )^2$$
也称为平方误差函数 Squared error function、Mean squared error

目的是 $\underset{\theta_0 , \theta_1}{minimize}J(\theta_0,\theta_1)$
# 3 Cost function intuition I
## 单一参数下代价函数与假设函数的关系
![IMAGE](/images/MachineLearning_AndrewNg_2/F4B776C812DDB22DF7102EDAAA8B8A3E.jpg)
简化为 $\theta_0 = 0$

![IMAGE](/images/MachineLearning_AndrewNg_2/316A35708F1C899F3CCF1AC03AFBA961.jpg)
对于(1,1)(2,2)(3,3)三个点，$h_\theta(x)=x$时正好经过三个点，此时$\theta_1 = 1$，$J(\theta_1) = 0$。

![IMAGE](/images/MachineLearning_AndrewNg_2/0B969E968913DA5B40B4BBF20E4D8BF7.jpg)
当$\theta_1 = 0.5$时，$J(\theta_1) = 0.58$。


![IMAGE](/images/MachineLearning_AndrewNg_2/5193979231F83D7C126B00D2EB3E41A0.jpg)
当$\theta_1 = 0$时，$J(\theta_1) = 2.3$。

![IMAGE](/images/MachineLearning_AndrewNg_2/7865D8C00D7C01E5776D62F012A24848.jpg)
代价函数最小时，$\theta_1 = 1$。

最好的假设函数是使得散点到该线的平均垂直距离最小的函数。理想情况下，该线应该通过训练数据集的所有点。 


# 4 Cost function intuition II
## 双参数下代价函数与假设函数的关系
$J(\theta_0,\theta_1)$函数大致如下图形状
<div align=center>
  <img width=465 src="https://vikingstudyhard.github.io/images/MachineLearning_AndrewNg_2/0AE007740ED8059A309EDBBEB6CF7E1B.jpg" >
</div>

转化为等高线图 contour plots/figures
同一条线上的点对应的$J(\theta_0,\theta_1)$值相同。
![IMAGE](/images/MachineLearning_AndrewNg_2/A68D3BF267AA32989234D55DE40793A8.jpg)
$\theta_0 = 800 $ $\theta_1 = -0.15$时，$ h_\theta (x)$ 如上图所示。

![IMAGE](/images/MachineLearning_AndrewNg_2/A6428B3E31F2C2ED50CED904124A04F6.jpg)
$\theta_0 = 360 $ $\theta_1 = 0$时，$ h_\theta (x)$ 如上图所示。


![IMAGE](/images/MachineLearning_AndrewNg_2/60EBCF577626A935F9FEC321BE324440.jpg)
等高线越往中心中心，$J(\theta_0,\theta_1)$值越小，对应的假设函数越好。


# 5 Gradient Descent 梯度下降
用梯度下降算法最小化代价函数J
已知函数 $J(\theta_0,\theta_1)$
想要 $\underset{\theta_0 ,\theta_1}{min}J(\theta_0,\theta_1)$


## 梯度下降的思路
- 给定 $\theta_0,\theta_1$ 初始值，通常初始化为$\theta_0 = 0,\theta_1 = 0$
- 调整$\theta_0,\theta_1$使$J(\theta_0,\theta_1)$减小，直到我们减小到一个希望的最小值

环顾四周，尽快下山要往哪个方向走？——找最佳的下山方向，即下降最快的方向
![IMAGE](/images/MachineLearning_AndrewNg_2/11946CF34AF9427A65A82096AD9ADAF9.jpg)
每一步都环顾四周找最佳的下山方向，直到收敛converge至局部最低点。

**不同的起始位置，可能得到不同的局部最优解local optimum**
![IMAGE](/images/MachineLearning_AndrewNg_2/0EA4FB989F19304321231160FEC88A12.jpg)

## 梯度下降的方法
repeat until covergence{
    $\quad \theta _j := \theta _j - \alpha \frac{\partial }{\partial \theta  _j}J(\theta_0,\theta_1)$ (for $j = 0$ and $j = 1$)
}

$a:=b$ assignment，赋值运算符，使a等于b值
$\alpha$ learning rate，学习率，控制梯度下降时迈出多大的步子，即上图中每个星号之间的距离。值很大时，梯度下降得迅速。

对于上式，需要同时更新 $\theta_0$ 和 $\theta_1$。
**simultaneous update正确方法：**
$temp0 := \theta _0 - \alpha \frac{\partial }{\partial \theta  _0}J(\theta_0,\theta_1)$
$temp1 := \theta _1 - \alpha \frac{\partial }{\partial \theta  _1}J(\theta_0,\theta_1)$
$\theta _0 := temp0$
$\theta _1 := temp1$

不正确方法：
$temp0 := \theta _0 - \alpha \frac{\partial }{\partial \theta  _0}J(\theta_0,\theta_1)$
$\theta _0 := temp0$
$temp1 := \theta _1 - \alpha \frac{\partial }{\partial \theta  _1}J(\theta_0,\theta_1)$
$\theta _1 := temp1$
没有做到同时更新，因为第三步计算 $temp1$ 时使用了新 的$\theta _0$。

# 6 Gradient Descent intuition 
## 导数项的意义
简化为只有一个参数 $\theta_1$：
Repeat until convergence{
     $\quad \theta _1 := \theta _1 - \alpha \frac{\partial }{\partial \theta  _1}J(\theta_1)$
}

$\alpha$ 学习率始终是正数。

1.当 $\theta _1$ 在对称轴右侧
![IMAGE](/images/MachineLearning_AndrewNg_2/835A08F5195B5A9CFBB4307AF812A3C5.jpg)

当 $\theta _1$ 在对称轴右侧，导数项 derivative term 即斜率 slope 为负数，因此新 $\theta _1$ 比之前小，向对称轴方向移动，最终收敛到函数的最小值。

2.当 $\theta _1$ 在对称轴左侧
![IMAGE](/images/MachineLearning_AndrewNg_2/9B517673316F8E106BBC84BBA438184E.jpg)

当 $\theta _1$ 在对称轴左侧，导数项即斜率为正数，因此新 $\theta _1$ 比之前大，向对称轴方向移动，最终收敛到函数的最小值。


## $\alpha$ 的意义
![IMAGE](/images/MachineLearning_AndrewNg_2/2D3855C9C6833650C417EEBF13B4B138.jpg)

如果 $\alpha$ 很小，梯度下降得慢。

如果 $\alpha$ 太大，可能一次次地越过最低点，会导致无法收敛甚至发散diverge。
当出现没有收敛或用了太多时间来获得最小值的情况时，意味着$\alpha$是错误的。

如果此时 $\theta _1$ 已经在一个局部最优点，下一步梯度下降会怎样？
<div align=center>
  <img width=585 src="https://vikingstudyhard.github.io/images/MachineLearning_AndrewNg_2/96EC02C8FF28F0AFEE43C3EDF51E8449.jpg" >
</div>

此时导数为 0 ，$\theta _1$不再改变。

即使 $\alpha$ 保持不变，梯度下降法也能收敛到局部最低点。

![IMAGE](/images/MachineLearning_AndrewNg_2/A2412BF9070241E428678E7797C242A4.jpg)
梯度下降的过程中，导数项自动变得越来越小，$\theta _1$更新的幅度也会越来越小。

# 7 Gradient Descent For Linear Regression 线性回归的梯度下降
## 公式推导
用梯度下降和代价函数结合得到线性回归算法，用直线模型拟合数据。
![IMAGE](/images/MachineLearning_AndrewNg_2/D7374612F612FE96DB8B3C1C9AC0F33F.jpg)
线性回归模型包括：线性假设linear hypothesis  以及 平方差代价函数 squared error cost function 

接下来要将梯度下降算法应用于最小化平方差代价函数。

$$ \begin{align*}\frac{\partial }{\partial \theta  _j}J(\theta_0,\theta_1)
 &= \frac{\partial }{\partial \theta  _j} \frac{1}{2m}\sum_{i=1}^{m}\left ( h_\theta (x^{(i)}) - y^{(i)} \right )^2\\
 &= \frac{\partial }{\partial \theta  _j} \frac{1}{2m}\sum_{i=1}^{m}\left ( \theta_0 + \theta_1x^{(i)}- y^{(i)} \right )^2\\
\end{align*}$$


$j = 0$ 时，$\frac{\partial }{\partial \theta  _0}J(\theta_0,\theta_1)=  \frac{1}{m}\sum_{i=1}^{m}\left ( h_\theta (x^{(i)}) - y^{(i)} \right )$


$j = 1$ 时，$\frac{\partial }{\partial \theta  _1}J(\theta_0,\theta_1)=  \frac{1}{m}\sum_{i=1}^{m}\left ( h_\theta (x^{(i)}) - y^{(i)} \right )\cdot x^{(i)}$

梯度下降算法为

repeat until covergence{
    $\quad \theta _0 := \theta _0 - \alpha \frac{1}{m}\sum_{i=1}^{m}\left ( h_\theta (x^{(i)}) - y^{(i)} \right )$
    $\quad \theta _1 := \theta _1 - \alpha \frac{1}{m}\sum_{i=1}^{m}\left ( h_\theta (x^{(i)}) - y^{(i)} \right )\cdot x^{(i)}$
}
注意同时更新$\theta _0 $$\theta _1 $。

## 可视化梯度下降过程
<div align=center>
  <img width=565 src="https://vikingstudyhard.github.io/images/MachineLearning_AndrewNg_2/69DB3FC7ED45D339810C543A4AB0D8FC.jpg" >
</div>

通常线性回归的代价函数是凸函数 convex function，没有很多局部最优解，只有一个全剧最优解。

初始化为$\theta _0=900$,$\theta _1=-0.1$
![IMAGE](/images/MachineLearning_AndrewNg_2/6B66672C792589971296FDBC8640B2C0.jpg)
此时假设函数为$h(x)=900-0.1x$
梯度下降到第二个点，假设函数也发生了变化：
![IMAGE](/images/MachineLearning_AndrewNg_2/731B07C8067BC6AF6B71C7A0BE4C4F5E.jpg)
梯度下降到第三个点：
![IMAGE](/images/MachineLearning_AndrewNg_2/EA8EA41E04D01BCE2B944EB65CDB99EA.jpg)
梯度下降到第四个点：
![IMAGE](/images/MachineLearning_AndrewNg_2/18C1EED4373417F265F9F4A4052E30B7.jpg)
……
最后到达全局最小值点：
![IMAGE](/images/MachineLearning_AndrewNg_2/26BB6DF735AC62BC2FFAADD02564C522.jpg)
得到了很好地拟合数据的假设函数，可以用来做预测。

## Batch 梯度下降
称为 Batch gradient descent：每一步梯度下降都遍历了整个训练集的样本，因为有这一项$\sum_{i=1}^{m}\left ( h_\theta (x^{(i)}) - y^{(i)} \right )$需要将所有预测值和实际值做差求和。

