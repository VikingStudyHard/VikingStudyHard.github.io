---
title: 【Andrew Ng】Machine Learning | 3 Linear Algebra review

date: 2019.01.19

categories: Machine learning

tags: 
 - Andrew Ng

description: 第三章 Linear Algebra review
mathjax: true
---
第三章 Linear Algebra review 回顾线性代数知识
# 1 Matrices and vectors 矩阵与向量
## 矩阵
Matrix: Rectangular array of numbers
Dimension of matrix: number of rows $\times$ number of columns

Matrix Elements 矩阵元素(entries of matrix)

$A=\begin{bmatrix}
1402 & 191 \\ 
 1371 &  821\\ 
 949 & 1437\\ 
 147 & 1448
\end{bmatrix}$
$\mathbb{R}^{4\times2}$ 表示所有 $4\times2$ 矩阵的集合 
$A_{ij}$=“$i,j$ entry” in the $i^{th}$ row, $j^{th}$ column.

## 向量
Vector: An $n \times  1$ matrix

$y=\begin{bmatrix}
460 \\ 
232 \\ 
315 \\ 
178
\end{bmatrix}$
4-dimensional vector 四维向量
$\mathbb{R}^{4}$ 表示所有四维向量的集合
$y_{i}=i^{th}$ element

1-indexed vector vs 0-indexed vector
1-下标向量 vs 0-下标向量
$y=\begin{bmatrix}
y_1 \\ 
y_2 \\ 
y_3 \\ 
y_4
\end{bmatrix}
y=\begin{bmatrix}
y_0 \\ 
y_1 \\ 
y_2 \\ 
y_3
\end{bmatrix}$

# 2 Addition and Scalar Multiplication 加法和标量乘法
## Matrix Addition and Subtraction 矩阵加减法
Addition and subtraction are element-wise, so you simply add or subtract each corresponding element:

$\begin{bmatrix}
1 & 0 \\ 
2 & 5 \\ 
3 & 1
\end{bmatrix}+
\begin{bmatrix}
4 & 0.5 \\ 
2 & 5 \\ 
0 & 1
\end{bmatrix}=
\begin{bmatrix}
5 & 0.5 \\ 
4 & 10 \\ 
3 & 2
\end{bmatrix}$

只能将相同维数的矩阵相加减，维数不同不能相加减。

## Scalar Multiplication 标量乘法
标量在这里指实数。
In scalar multiplication, we simply multiply every element by the scalar value:
$3\times
\begin{bmatrix}
1 & 0  \\ 
2 & 5 \\ 
3 & 1
\end{bmatrix}=
\begin{bmatrix}
3 & 0 \\ 
6 & 15 \\ 
9 & 3
\end{bmatrix}$

In scalar division, we simply divide every element by the scalar value:
$\begin{bmatrix}
4 & 0  \\ 
6 & 3 
\end{bmatrix}/4=
\begin{bmatrix}
1 & 0 \\ 
1.5 & 0.75
\end{bmatrix}$

# 3 Matrix-vector multiplication
We map the column of the vector onto each row of the matrix, multiplying each element and summing the result.

$\begin{bmatrix}
1 & 2 & 1 &5 \\ 
0 & 3 & 0 & 4 \\ 
-1 & -2 & 0 & 0
\end{bmatrix}\times
\begin{bmatrix}
1  \\ 
3  \\ 
2 \\ 
1  
\end{bmatrix}=
\begin{bmatrix}
14 \\ 
13 \\ 
-7
\end{bmatrix}$

The result is a vector. The number of columns of the matrix must equal the number of rows of the vector.

An m x n matrix multiplied by an n x 1 vector results in an m x 1 vector.

> e.g.house sizes：2104,1416,1534,852.$h_\theta(x)=-40+0.25x$
可以根据构建矩阵，根据假设函数构建向量，两者相乘得到对每个房子的预测价格。
$\begin{bmatrix}
1 & 2104  \\ 
1 & 1416  \\ 
1 & 1534  \\ 
1 & 852 
\end{bmatrix} \times
\begin{bmatrix}
-40 \\ 
0.25
\end{bmatrix}=
\begin{bmatrix}
-40 \times 1 + 0.25\times 2104\\
-40 \times 1 + 0.25\times 1416\\ 
-40 \times 1 + 0.25\times 1534\\ 
-40 \times 1 + 0.25\times 852
\end{bmatrix}=
\begin{bmatrix}
h_\theta(2104)\\
h_\theta(1416)\\ 
h_\theta(1534)\\ 
h_\theta(852)
\end{bmatrix}
$
用$prediction = DataMatrix \times parameters$可以简化计算过程，一个公式就能得到所有结果，代码简洁效率高，不用for循环一个一个算。


# 4 Matrix-Matrix Multiplication
矩阵与矩阵相乘来解决 $\theta_0$ $\theta_1$ 参数的计算问题，而不需要梯度下降这种迭代算法
An m x n matrix multiplied by an n x o matrix results in an m x o matrix. 

To multiply two matrices, the number of columns of the first matrix must equal the
number of rows of the second matrix.



> e.g.house sizes：2104,1416,1534,852.
有三个假设函数
1.$h_\theta(x)=-40+0.25x$
2.$h_\theta(x)=200+0.1x$
3.$h_\theta(x)=-150+0.4x$
可以根据构建矩阵，根据假设函数构建向量，两者相乘得到对每个房子的预测价格。
$\begin{bmatrix}
1 & 2104  \\ 
1 & 1416  \\ 
1 & 1534  \\ 
1 & 852 
\end{bmatrix} \times
\begin{bmatrix}
-40 & 200 & -150\\ 
0.25 & 0.1 & 0.4
\end{bmatrix}=
\begin{bmatrix}
1h_\theta(2104) & 2h_\theta(2104) & 3h_\theta(2104)\\
1h_\theta(1416) & 2h_\theta(1416) & 3h_\theta(1416)\\ 
1h_\theta(1534) & 2h_\theta(1534) & 3h_\theta(1534)\\ 
1h_\theta(852)  & 2h_\theta(852)  & 3h_\theta(852) 
\end{bmatrix}
$
可对众多假设进行预测





# 5 Matrix Multiplication Properties
Matrices are not commutative:  $A\times B \neq B\times A$ 矩阵和矩阵相乘没有交换律
Matrices are associative:  $(A\times B)\times C=A\times (B \times C)$ 矩阵和矩阵相乘服从结合律

Identity matrix 单位矩阵，定义为  $\mathit{I}$ 或 $\mathit{I}_{n\times n}$
如：
$\begin{bmatrix}
1 & 0 \\ 
0 & 1 
\end{bmatrix}$    $2\times 2$    $\begin{bmatrix}
1 & 0 & 0 \\ 
0 & 1 & 0 \\
0 & 0 & 1
\end{bmatrix}$    $3\times 3$    $\begin{bmatrix}
1 & 0 & 0 & 0\\ 
0 & 1 & 0 & 0\\
0 & 0 & 1 & 0\\
0 & 0 & 0 & 1
\end{bmatrix}$    $4\times 4$

对任何矩阵 $A$，有 $A \cdot I = I \cdot A = A$

# 6 Inverse and Transpose
## Inverse 逆运算
The inverse of a matrix $A$ is denoted $A^{−1}$. Multiplying by the inverse results in the identity matrix.

$AA^{−1} = A^{−1}A = I$

只有$m\times m$ 的矩阵——方矩阵 square matrix、且不能都为0 的矩阵才有其逆矩阵。

没有逆矩阵的矩阵称为 singular 奇异矩阵、degenerate 退化矩阵
计算逆矩阵：
Octave  pinv(A) 
Matlab  inv(A) 
## Transpose 转置运算

$A = \begin{bmatrix}
1 & 2 & 0 \\ 
3 & 5 & 9 
\end{bmatrix}$$A^T = \begin{bmatrix}
1 & 3 \\ 
2 & 5 \\
0 & 9
\end{bmatrix}$
$A_{ij}=A^T_{ji}$