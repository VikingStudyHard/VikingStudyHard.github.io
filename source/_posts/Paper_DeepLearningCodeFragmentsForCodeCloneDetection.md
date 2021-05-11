---

title: 【论文精读】Deep learning code fragments for code clone detection

date: 2019.7.13

categories: 
 - [代码克隆]

tags: 
 - Code Clone
 - 论文精读

description: 第一个为克隆检测提出语言模型和embedding
mathjax: true

---
> White M, Tufano M, Vendome C, et al. Deep learning code fragments for code clone detection[C]//Proceedings of the 31st IEEE/ACM International Conference on Automated Software Engineering. ACM, 2016: 87-98.

# 做什么
提出一种全新的方法来检测代码克隆。
# 怎么做
将term映射到特征空间中的连续值向量，并用距离衡量相似性，同时将 AST 转化为 full 二叉树，根据叶子结点对identifiers和literal types进行操作，从词法和句法两个层面提取代码特征。（extract source code features at both lexical and syntax levels.）

{% img https://vikingstudyhard.github.io/images/Paper_DeepLearningCodeFragmentsForCodeCloneDetection/361126D68F34A6C2C00F19A2B67D914A.jpg 465 %}
## Lexical Level 
语言模型 RtNN
{% img https://vikingstudyhard.github.io/images/Paper_DeepLearningCodeFragmentsForCodeCloneDetection/C2EAAA7C6FC6E2E01081038A72924D16.jpg 367 %}
白点是 one-hot 的 term 向量，黑点是 continuous-valued 状态，灰点是后验分布。

当前term是$t(i)$，先前状态是$ z(i-1)$。
输入层为：
$$
x(i)=[t(i) ; z(i-1)]
$$
隐藏层为：
$$
z(i)=f(\alpha t(i)+\beta z(i-1))
$$
输出层为：
$$
y(i)=p(t | x(i))=\operatorname{softmax}(\gamma z(i))
$$
用 $\operatorname{argmax}_{k} y_{k}(i)$ 预测代码行中的下一项term。
模型 $\theta=\{\alpha, \beta, \gamma\}$：嵌入矩阵 $\alpha$ 的每列对应一个term。
## Syntactic Level
**完整的二叉树到Olive Trees**

考虑语句`int foo = 42;`。该语句的AST是图3中(1)-(5)中描述的完整二叉树。
{% img https://vikingstudyhard.github.io/images/Paper_DeepLearningCodeFragmentsForCodeCloneDetection/1AA3F3F7DBEA51F880CF9AA05B985145.jpg 180 %}

假设每个AST节点都有一个特殊的属性repr，例如，2.repr存储图3中SimpleName(2)的表示。

我们通过使用其词法元素，来为每个叶子结点初始化该属性，以选择嵌入矩阵 $\alpha$ 中的相应列。例如，如果词汇元素`int`映射到 $\alpha$ 的第j列，则初始化图3中的PrimitiveType(1)的repr，使得$1.repr= \alpha·j$。


对于非叶子结点，例如图3中的VariableDeclarationFragment(4)和VariableDeclarationStatement(5)，此属性初始化为null。此时，我们使用了在词汇级别挖掘的模式来初始化序列的embedding。接下来，我们使用自动编码器来组合嵌入。自动编码器的规范形式是具有一个输入层$x$，一个隐藏层$z$和一个输出层$y$的神经网络。
$$
\begin{array}{l}{z=g\left(\varepsilon x+\beta_{z}\right)} \\ {y=h\left(\delta z+\beta_{y}\right)}\end{array}
$$

其中 $ \varepsilon=\left[\varepsilon_{\ell}, \varepsilon_{r}\right] \in \mathbb{R}^{n \times 2n}$是 $\varepsilon$ncoder；$\delta=\left[\delta_{\ell} ; \delta_{r}\right] \in  \mathbb{R}^{2n \times n}$ 是 $\delta$ecoder；$\beta_{z} \in \mathbb{R}^{n} $ 且 $ \beta_{y} \in \mathbb{R}^{2 n} $ 是 $\beta$iases。


促使AST转换为完整的二叉树：自动编码器的输入是两个兄弟节点代码的矢量，即 $x=\left[x_{\ell} ; x_{r}\right] \in \mathbb{R}^{2 n}$。例如，为了计算图3中VariableDeclarationFragment(4)的表示，我们将$x = [2.repr; 3.repr]$呈现给模型。限制隐藏层的大小（即$| z | = n <2n）$强制模型学习其输入的压缩表示。这种压缩，在方程式中为$z$，作为我们存储在非叶子节点的repr属性中的挖掘表示。本质上，模型将输入嵌入到较低维度的特征空间中，就像嵌入一个热词矢量的语言模型一样。换句话说，语言模型将词法元素转换为embedding，并且自动编码器将任何两个嵌入压缩为具有与术语嵌入相同维度的向量。输出$y=\left[\hat{x}_{\ell} ; \hat{x}_{r}\right] \in \mathbb{R}^{2 n}$被称为模型对输入的重建。训练模型涉及测量原始输入向量与重建之间的距离：
$$
E\left(x_{\ell}, x_{r} ; \varepsilon, \delta, \beta_{z}, \beta_{y}\right)=\left\|x_{\ell}-\hat{x}_{\ell}\right\|_{2}^{2}+\left\|x_{r}-\hat{x}_{r}\right\|_{2}^{2}
$$
如果模型可以有效地学习输入的区分特征，那么它将能够概括并忠实地重建从域中采样的任何输入向量。

刚刚演示了传统自动编码器如何压缩两个词法元素的适度序列，但为了支持克隆检测，我们学习了更多的代码。由于树中每个节点的代码具有相同的大小，我们可以递归地应用自动编码器（RvNN）来对不同级别的完整二叉树进行建模。我们用于压缩图3中的SimpleName（2）和NumberLiteral（3）的自动编码器可以递归应用，只要VariableDeclarationFragment（4）的代码与PrimitiveType（1）的代码合并并呈现给用于计算VariableDeclarationStatement（5）的代码的相同模型：$5 . \mathrm{repr}=g\left(\left[\varepsilon_{\ell}, \varepsilon_{r}\right][1 .\mathrm{repr}; 4 .\mathrm{repr}]+\beta_{z}\right)$。如前所述，为了训练模型，我们解码表示（即，$y=h\left(\left[\delta_{\ell} ; \delta_{r}\right][5 . \mathrm{repr}]+\beta_{y}\right)$）并将重建与输入进行比较（即，$x=[1 . \mathrm{repr};  4 .\mathrm{repr} ] )$）来衡量权重。但是现在误差是所有重建误差的（加权）总和，其中较大的编程结构将对形成片段的表示具有更大的影响。例如，VariableDeclarationFragment（4）对调优5.repr的影响大于PrimitiveType（1）。在前向传递中计算每个非终结节点的代码之后，反向传播结构算法计算（全局）误差函数相对于模型组件的偏导数。然后使用标准方法优化误差信号。一旦深度学习者在一些时期之后收敛，我们就用完整的二元树来表示生成一棵橄榄树。