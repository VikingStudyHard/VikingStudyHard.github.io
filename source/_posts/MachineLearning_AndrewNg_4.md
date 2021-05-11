---
title: 【Andrew Ng】Machine Learning | 4 LinearRegression with multiple variables

date: 2019.01.20

categories: Machine learning

tags: 

 - Andrew Ng

description: 第四章 LinearRegression with multiple variables
mathjax: true
---

第四章 LinearRegression with multiple variables 
# 1 Multiple Features 多特征量

<div align=center>
  <img width=700 src="https://vikingstudyhard.github.io/images/MachineLearning_AndrewNg_4/3ED9C7B973E1F2C6541549548F5F4205.jpg" >
</div>

$m$ = the number of training examples 样本数量
$n$ = the number of features 上图中 $n=4$
$x^{(i)}$ = the input (features) of the $i^{th}$ training example 
上图中 $x^{(2)}=\begin{bmatrix}
1416 \\ 
3 \\ 
2 \\ 
40
\end{bmatrix}$
$x^{(i)}_j$ = value of feature $j$ in the $i^{th}$ training example 第$i$个样本的第$j$个特征
上图中 $x^{(2)}_3 = 2$ 


假设函数变成：$h_\theta(x)=\theta_0+\theta_1x_1+\theta_2x_2+\theta_3x_3+\theta_4x_4+...+\theta_nx_n$

定义$x_0=1$，则$x^{(i)}_0 = 1$，那么特征向量$x=\begin{bmatrix}
x_0 \\ 
x_1 \\ 
x_2 \\ 
\vdots  \\
x_n 
\end{bmatrix}\in \mathbb{R}^{n+1}$，参数向量$\theta=\begin{bmatrix}
\theta_0 \\ 
\theta_1 \\ 
\theta_2 \\ 
\vdots \\
\theta_n 
\end{bmatrix}\in \mathbb{R}^{n+1}$

假设函数可写为:



$$\begin{align*} h_\theta(x) &=\theta_0x_0+\theta_1x_1+\theta_2x_2+\theta_3x_3+\theta_4x_4+...+\theta_nx_n\\
 &= \theta^Tx \\
 &= \begin{bmatrix} \theta_0 &\theta_1 & \theta_2 & ...  & \theta_n 
 \end{bmatrix} \begin{bmatrix}
x_0 \\ 
x_1 \\ 
x_2 \\ 
\vdots  \\
x_n 
 \end{bmatrix}
\end{align*}$$



Multivariate linear regression 多元线性回归

# 2 Gradient descent for multiple variables

多元梯度下降
Hypothesis: $h_\theta(x)=\theta^Tx=\theta_0x_0+\theta_1x_1+\theta_2x_2+\theta_3x_3+\theta_4x_4+...+\theta_nx_n$

Parameters: $\theta=\begin{bmatrix}
\theta_0 \\ 
\theta_1 \\ 
\theta_2 \\ 
\vdots  \\
\theta_n 
\end{bmatrix}$

Cost function: $J(\theta)=\frac{1}{2m}\sum_{i=1}^{m}\left ( h_\theta (x^{(i)}) - y^{(i)} \right )^2$

Gradient descent: 
Repeat{
    $\quad \theta _j := \theta _j - \alpha \frac{\partial }{\partial \theta  _j}J(\theta)$
}
对于 $j=0,...,n$ 同时更新
## 只有一个特征值
$n=1$时
Repeat{
$\quad \theta _0 := \theta _0 - \alpha \underbrace{\frac{1}{m}\sum_{i=1}^{m}\left ( h_\theta (x^{(i)}) - y^{(i)} \right )}_{\frac{\partial }{\partial \theta  _0}J(\theta)}$
$\quad \theta _1 := \theta _1 - \alpha \frac{1}{m}\sum_{i=1}^{m}\left ( h_\theta (x^{(i)}) - y^{(i)} \right )\cdot x^{(i)}$
(同时更新$\theta_0$ $\theta_1$)
}

## 不止一个特征值
$n\geq1$时
Repeat{
$\quad \theta _j := \theta _j - \alpha \frac{1}{m}\sum_{i=1}^{m}\left ( h_\theta (x^{(i)}) - y^{(i)} \right )\cdot x^{(i)}_j$
(同时更新$\theta_j$ for $j=0,...,n$)
}

$\theta _0 := \theta _0 - \alpha \frac{1}{m}\sum_{i=1}^{m}\left ( h_\theta (x^{(i)}) - y^{(i)} \right )\cdot x^{(i)}_0$
$\theta _1 := \theta _1 - \alpha \frac{1}{m}\sum_{i=1}^{m}\left ( h_\theta (x^{(i)}) - y^{(i)} \right )\cdot x^{(i)}_1$
$\theta _2 := \theta _2 - \alpha \frac{1}{m}\sum_{i=1}^{m}\left ( h_\theta (x^{(i)}) - y^{(i)} \right )\cdot x^{(i)}_2$
...

# 3 Gradient Descent in Practice I - Feature Scaling 特征缩放
确保features在相似的范围内，使梯度下降更快地收敛
> E.g. 
$x_1$ = size (0-2000 feet$^2$)
$x_2$ = number of bedrooms (1-5)

<div align=center>
  <img width=348 src="https://vikingstudyhard.github.io/images/MachineLearning_AndrewNg_4/46E129573AB8B4C4BB4A7EE65CA8C0F1.jpg" >
</div>

梯度下降反复来回振荡，需要花很长时间找到全局最小值。
> E.g.
$x_1 =\frac{ size (feet^2)}{2000}$
> 
> $x_2 =\frac{number\ of\ bedrooms}{5}$


<div align=center>
  <img width=262 src="https://vikingstudyhard.github.io/images/MachineLearning_AndrewNg_4/EBB8049827DD074604F14C3C11778088.jpg" >
</div>

使 $0\leq x_i\leq1$，收敛更快。

将每个feature的取值约束到 $-1\leq x_i\leq1$

Mean normalization 均值归一化
用 $x_i-\mu_i$ 代替 $x_i$，使特征值的平均值约为$0$，但是不应用于 $x_0$，因为$x_0$恒等$1$。
> E.g.
$x_1 =\frac{ size - 1000 }{2000}$
> 
> $x_2 =\frac{\\#bedrooms-2}{5}$

$-0.5\leq x_i\leq0.5$，收敛更快。

两者整合起来就是：
$$x_i:=\frac{x_i- \mu_i}{s_i}$$
其中，$\mu_i$ 是训练集中$x_i$的平均值，${s_i}$是该特征值的范围，即（max -
min），或者是标准差standard deviation。

# 4 Gradient Descent in Practice II - Learning rate 学习率

## 如何确保梯度下降是正确工作的
### Debugging gradient descent
作图
<div align=center>
  <img width=483 src="https://vikingstudyhard.github.io/images/MachineLearning_AndrewNg_4/4E5612347FE5DAF3749E19D6699AC4BC.jpg" >
</div>

横轴代表的是迭代次数，而不是$\theta$。
曲线上的点表示：用某个迭代次数后得到的$\theta$，计算$J(\theta)$的值。
正常情况下：**每一步迭代后，$J(\theta)$都应该下降**，例如图中300到400迭代次数中，曲线平坦，也就是说差不多已经收敛了。

### Automatic convergence test
自动收敛测试，判断是否已经收敛。
如果代价函数$J(\theta)$一步迭代后的下降小于一个很小的值$\varepsilon$，就可以判断已经收敛。$\varepsilon$可以是$10_{-3}$，通常选择一个合适的阈值$\varepsilon$很困难。所以一般都用上面的方法。

## 如何选择学习率 $\alpha$
<div align=center>
  <img width=530 src="https://vikingstudyhard.github.io/images/MachineLearning_AndrewNg_4/CEE886A17ECBE20DC6D1CE233E9354B2.jpg" >
</div>

如果$J(\theta)$的值反而增加了，或者一会增加一会减少，那就应该使用较小的$\alpha$。
但如果$\alpha$过小，收敛的过程会特别慢。

$\alpha$可以尝试为 ...0.001,0.003,0.01, 0.03,0.1,0.3,1,...然后绘制$J(\theta)$随迭代次数变化的曲线。


# 5 Features and Polynomial Regression 特征与多项式回归
We can **combine** multiple features into one. 
For example, we can combine $x_1$ and $x_2$ into a new feature $x_3$ by taking   $x_1\cdot  x_2 $.


## Housing prices prediction
$h_\theta(x)=\theta_0+\theta_1\times frontage + \theta_2\times depth$
房子的临街宽度和垂直宽度
<div align=center>
  <img width=595 src="https://vikingstudyhard.github.io/images/MachineLearning_AndrewNg_4/C6927DD0D2F8BE99E48605A63EF3AD08.jpg" >
</div>


可以直接将面积作为新的特征，得到更好的模型。

quadratic model 二次模型 过了对称轴会下降 不适合 --> cubic model 三次模型
<div align=center>
  <img width=663 src="https://vikingstudyhard.github.io/images/MachineLearning_AndrewNg_4/1E8B7DE2D8B45117DA486E8E515A456F.jpg" >
</div>


要找到合适的模型，并且进行特征缩放，使值的范围变得具有可比性。

或者$h_\theta(x)=\theta_0+\theta_1(size) + \theta_2 \sqrt{(size)}$
函数到后来不会下降，而是趋于平缓。

# 6 Linear regression with multiple variables -- Normal equation 正规方程

正规方程：对于某些线性回归问题，有更好的方法求参数 $\theta$的最优值。

梯度下降法(gradient descent)中为了最小化代价函数 $J(\theta)$，使用迭代算法(iterative algorithm)，经过许多步迭代，收敛(converge)到全局最小值。

正规方程可以一次直接求解得到$\theta$的最优值，而不用多次迭代。

## Intuition
当$\theta$是实数，导数为0时$\theta$的值即为$\theta$的最优值

<div align=center>
  <img width=757 src="https://vikingstudyhard.github.io/images/MachineLearning_AndrewNg_4/47A67903FC373617014D2E7E42F3A32D.jpg" >
</div>


当$\theta$是n+1维的参数向量，需要逐个对参数$\theta_j$求$J(\theta)$的偏导数，并全部置0，求出所有$\theta$的值。
<div align=center>
  <img width=771 src="https://vikingstudyhard.github.io/images/MachineLearning_AndrewNg_4/E3C1A1C5B4575F68DE05B84A1B8C57F3.jpg" >
</div>


这样做很复杂。


## 例子
<div align=center>
  <img width=762 src="https://vikingstudyhard.github.io/images/MachineLearning_AndrewNg_4/1BB4DFE1C7EB5F75BD84B8E11C29404C.jpg" >
</div>

为了实现正规方程法，需要加上$x_0$，取值永远是1。通过训练样本的所有特征变量建立 $m\times (n+1)$ 的矩阵  $X$，通过所有y值建立 $m$ 维向量 $y$。其中$m$是训练样本数量，$n$是特征变量的数量。

<div align=center>
  <img width=800 src="https://vikingstudyhard.github.io/images/MachineLearning_AndrewNg_4/2A6D6478D852BE0ECED285D90893991E.jpg" >
</div>


计算$$\theta=(X^TX)^{-1}X^Ty$$
> X transpose X  inverse times X transpose y

可以得到使代价函数最小化的$\theta$

## design matrix 设计矩阵
有$m$个样本：$(x^{(1)},y^{(1)}),...,(x^{(m)},y^{(m)})$
有$n$个特征：$x{(i)}=\begin{bmatrix}
x^{(i)}_0 \\ 
x^{(i)}_1 \\ 
x^{(i)}_2 \\ 
...\\
x^{(i)}_n
\end{bmatrix} \in \mathbb{R}^{(n+1)}$

则 $X=\begin{bmatrix}
---(x^{(1)})^T--- \\ 
---(x^{(2)})^T--- \\ 
---(x^{(3)})^T--- \\ 
\vdots \\
---(x^{(m)})^T---
\end{bmatrix}$

> E.g.
若$x{(i)}=\begin{bmatrix}
1 \\
x^{(i)}_1
\end{bmatrix} $，则 $X=\begin{bmatrix}
1, x^{(1)}_1 \\ 
1, x^{(2)}_1 \\ 
1, x^{(3)}_1 \\ 
\vdots \\
1, x^{(m)}_1
\end{bmatrix}$ ，$y=\begin{bmatrix}
y^{(1)} \\ 
y^{(2)} \\ 
y^{(3)} \\ 
\vdots \\
y^{(m)}
\end{bmatrix}$
然后计算$\theta=(X^TX)^{-1}X^Ty$即可。

Octave
```
pinv(X'*X)*X'*y
```

其中，X'为转置，pinv为求逆函数。

使用正规方程不需要特征缩放(feature scaling)。
##  梯度下降和正规方程的优缺点

- Gradient Descent
  + 缺点
    + 需要选择学习速率 $\alpha$
    + 需要多次迭代
  + 优点
    + $n$ 很大（特征变量很多）时也会运行很好

- Normal Equation
  + 优点
    + 不需要选择学习速率 $\alpha$
    + 不需要迭代   
  + 缺点
    + 需要计算 $(X^TX)^{-1}$ ，$X^TX$是$n\times n$的矩阵，而求逆矩阵的代价以矩阵维度的三次方增长，即$O(n^3)$
    + $n$ 很大（特征变量很多）时会很慢


所以，$n$ 很大（$n=10^6$ 以上）时用梯度下降法；$n$ 很小时（n为几万以下），用正规方程。

# 7 Linear regression with multiple variables -- Normal equation and non-invertibility 
正规方程以及不可逆性

$X^TX$不可逆怎么办
不可逆矩阵 non-invertible matrices：奇异矩阵 singular matrices / 退化矩阵 degenerate matrices


## $X^TX$不可逆的两种原因
- 冗余特征(线性相关)
  e.g. $x_1=size \  in \  feet^2$, $x_2=size \  in  \  m^2$
    $1\ m = 3.28\ feet$，即$x_1 = 3.28^2\ x_2$，可以删除其中一个特征
- 特征过多(eg.$m\leq n$)
  - 删除一些特征，或正则化(regularization) 