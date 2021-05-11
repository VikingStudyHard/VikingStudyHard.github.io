---

title: 【论文精读】Neural Network-based Graph Embedding for Cross-Platform Binary Code Similarity Detection

date: 2019.7.14

categories: 
 - [代码克隆]

tags: 
 - Code Clone
 - 论文精读

description: 第一个基于神经网络的方法来生成binary function的embedding
mathjax: true

---
> Xu X, Liu C, Feng Q, et al. Neural network-based graph embedding for cross-platform binary code similarity detection[C]//Proceedings of the 2017 ACM SIGSAC Conference on Computer and Communications Security. ACM, 2017: 363-376.

# 做什么
跨平台二进制代码相似性检测。基于深度神经网络的方法来生成用于相似性检测的二进制函数的embedding
# 怎么做
提出了一种新的基于神经网络的方法来计算embedding，即数值向量，基于每个二元函数的控制流图，然后可以通过测量两个函数的embedding之间的距离，有效地进行相似性检测。

基于神经网络的嵌入生成：使用神经网络将ACFG转换为嵌入，以克服先前基于图匹配的方法的局限性。
带有属性的控制流图(attributed control flow graph, ACFG)
## 代码相似度嵌入问题
我们假设存在一个$oracle$ $\pi$确定已知任务的代码相似性度量。已知两个二进制程序函数$f_1$，$f_ 2$，$\pi(f_1,f_2)= 1$表示它们相似;否则，$π(f_1,f_2)=-1$表示它们不相似。

代码相似性嵌入问题的目的是找到映射$\phi$，将函数$f$的 ACFG 映射到矢量表示$\mu$。给定易于计算的相似度函数 $Sim(·,·)$和两个二进制程序函数$f_1$，$f_ 2$，当$π(f_1,f_2)=-1$，$\operatorname{Sim}\left(\phi\left(f_{1}\right), \phi\left(f_{2}\right)\right)$很大。


方法 $f$的二进制代码用$ACFG$ $g$表示，embedding mapping $\phi$是神经网络，输入是$ACFG$，训练 graph embedding network 用于相似性检测问题，即训练$\phi$使其在区分两个输入的ACFG之间的相似性方面做得好。

Siamese架构将两个方法作为输入，并将相似性得分作为输出。Siamese架构嵌入了graph embedding network：Structure2vec。即输入为graph对$g1$和$g2$，输出为$\pi\left(f_{1}, f_{2}\right)$


## graph embedding network

### Structure2vec

将ACFG表示为$g=\langle\mathcal{V}, \mathcal{E}\rangle$，其中$\mathcal{V}$和$\mathcal{E}$分别是顶点和边的集合;此外，图中的每个顶点$v$具有对应于ACFG中的  basic-block 特征的附加特征$x_v$。图形嵌入网络将首先计算每个顶点$v \in \mathcal{V}$的$p$维特征$\mu_{v}$，然后将$g$的嵌入向量$\mu_{g}$计算为这些顶点embedding的聚合。即$\mu_{g} :=A_{v \in \mathcal{V}}\left(\mu_{v}\right)$，其中$A$是聚合函数，即求和或平均值。在这项工作中，我们选择$\mu_{g}=\sum_{v \in \mathcal{V}}\left(\mu_{v}\right)$。

#### Basic Structure2vec Approach
Structure2vec受图形模型推理算法的启发，其中顶点的特定特征$x_v$根据图形拓扑$g$结构递归地聚合。
$\mathcal{N}(v)$ 表示在图中顶点的所有相邻节点的集合。

Structure2vec网络的一个变体将每个顶点处的embedding $\mu_{v}^{(0)}$ 初始化为 $0$，并在每次迭代时将 embedding 更新为：
$$
\mu_{v}^{(t+1)}=\mathcal{F}\left(x_{v}, \sum_{u \in \mathcal{N}(v)} \mu_{u}^{(t)}\right), \forall v \in \mathcal{V}
$$

$\mathcal{F}$是一个通用的非线性映射。embedding的更新过程是基于图形拓扑，并以同步的方式进行。只有在上一轮的所有顶点的 embedding 更新完成后，才会开始扫描顶点做新一轮嵌入。顶点特征$x_v$通过非线性传播函数 $\mathcal{F}$ 传播到其他顶点。此外，执行更新的迭代越多，顶点特征将传播到远距离顶点，并在远端顶点非线性聚合。最后，如果在$T$次迭代之后终止更新过程，则每个顶点嵌入 $\mu_{v}^{(T)}$ 将包含关于它的T-hop邻域的信息，这些信息是由图拓扑和所涉及顶点的特征确定的。

### $\mathcal{F}$的参数化

{% img https://vikingstudyhard.github.io/images/Paper_Gemini/A6263719B36DC7F1C7CC27EEB1B0A574.jpg 280 %}
$\mathcal{F}$的公式如下：
$$
\mathcal{F}\left(x_{v}, \sum_{u \in \mathcal{N}(v)} \mu_{u}\right)=\tanh \left(W_{1} x_{v}+\sigma\left(\sum_{u \in \mathcal{N}(v)} \mu_{u}\right)\right)
$$
其中 $x_v$ 是用于图节点（或basic-block）级特征的d维向量，$W_1$是$d\times p$矩阵，并且$p$是如上所述的嵌入大小。为了使非线性变换 $\sigma(\cdot)$ 更强大，我们将$\sigma$本身定义为n层全连通神经网络：
$$
\sigma(l)=\underbrace{P_{1} \times \operatorname{ReLU}\left(P_{2} \times \ldots \operatorname{ReLU}\left(P_{n} l\right)\right)}_{n \text { levels }}
$$
其中$P_i(i = 1,...,n)$是 $p\times p$ 矩阵。我们将$n$称为嵌入深度embedding depth。这里，ReLU是rectified linear unit，即 $ReLU(x)=max\{0，x\}$。

我们对更新函数$\mathcal{F}$的新颖参数化以及第3.5节中描述的迭代更新方案完成了ACFG的嵌入网络。

在算法1中总结了用于为每个ACFG生成嵌入的整体算法。在该算法中，$W_2$ 是另一个用于变换嵌入向量的 $p\times p$ 矩阵。我们将其输出表示为 $\phi(g)$。
{% img https://vikingstudyhard.github.io/images/Paper_Gemini/8F8B21724102F8297A5BBE578C657703.jpg 460 %}



{% img https://vikingstudyhard.github.io/images/Paper_Gemini/E2BA6B1F9B72CA0F6DD77A5C7503C073.jpg 546 %}

## 使用Siamese架构学习参数
Siamese架构使用两个相同的图形嵌入网络，即Structure2vec，它们在顶部连接。每个图形嵌入网络将采用一个ACFG $g_i(i = 1,2)$作为其输入，并输出嵌入 $\phi\left(g_{i}\right)$。Siamese架构的最终输出是两个嵌入的余弦距离。此外，两个嵌入网络共享同一组参数，因此在训练期间两个网络保持相同。整体架构如图4所示。
{% img https://vikingstudyhard.github.io/images/Paper_Gemini/1A0DAAFCFCF2E262B7C10A0044A48A83.jpg 298 %} 

给定一组$K$对ACFG $\left\langle g_{i}, g_{i}^{\prime}\right\rangle$，具有基础真值(ground truth)配对信息$y_{i} \in\{+1,-1\}$，其中$y_{i} = +1$表示 $ g_{i}$ 和 $g_{i}^{\prime}$ 相似，即 $\pi\left(g_{i}, g_{i}^{\prime}\right)=1$，否则为 $y_i = -1$。我们将每对的Siamese网络输出定义为：
$$
\operatorname{Sim}\left(g, g^{\prime}\right)=\cos \left(\phi(g), \phi\left(g^{\prime}\right)\right)=\frac{\left\langle\phi(g), \phi\left(g^{\prime}\right)\right\rangle}{\|\phi(g)\| \cdot\left\|\phi\left(g^{\prime}\right)\right\|}
$$

其中$\phi(g)$由算法1产生。然后训练模型参数$W_1，P_1，...，P_n$和$W_2$，我们将优化以下目标函数:
$$
\min _{W_{1}, P_{1}, \ldots, P_{n}, W_{2}} \sum_{i=1}^{K}\left(\operatorname{Sim}\left(g_{i}, g_{i}^{\prime}\right)-y_{i}\right)^{2}
$$

我们可以用随机梯度下降来优化上述公式。根据图形拓扑递归地计算参数的梯度。最后，一旦Siamese网络可以实现良好的性能（例如，使用AUC作为度量），训练过程终止，并且训练的图形嵌入网络可以将输入图形转换为可用于相似性检测的有效嵌入。

# 两种图形嵌入
graph embedding

1.embed the nodes of a graph: find a map from the nodes to a vector space, so that the structural information of the graph is preserved
2.find a embedding vector that represents the whole graph



# 三篇论文比较
> 
Li L, Feng H, Zhuang W, et al. Cclearner: A deep learning-based clone detection approach[C]//2017 IEEE International Conference on Software Maintenance and Evolution (ICSME). IEEE, 2017: 249-260.
White M, Tufano M, Vendome C, et al. Deep learning code fragments for code clone detection[C]//Proceedings of the 31st IEEE/ACM International Conference on Automated Software Engineering. ACM, 2016: 87-98.
Xu X, Liu C, Feng Q, et al. Neural network-based graph embedding for cross-platform binary code similarity detection[C]//Proceedings of the 2017 ACM SIGSAC Conference on Computer and Communications Security. ACM, 2017: 363-376.


共同点：都是基于深度学习的克隆检测工具

不同点：
- 原理上：
    - CCLEARNER：第一个使用深度学习的基于token的克隆检测方法。通过ANTLR lexer 和 Eclipse ASTParser 得到token，将出现的token和每个 token 的出现次数组成 token-frequency list，将token分为八类并创建 token-frequency lists 的向量，用向量之间的相似性来反映相似性。
    - White：第一个为克隆检测提出语言模型和embedding。与基于token不同，此方法将term映射到特征空间中的连续值向量，并用距离衡量相似性，同时将 AST 转化为 full 二叉树，根据叶子结点对identifiers和literal types进行操作，从词法和句法两个层面提取代码特征。（extract source code features at both lexical and syntax levels.）
    - Gemini：第一个基于神经网络的方法来生成binary function的embedding。将方法的二进制代码用ACFG表示，通过graph embedding network，基于图形拓扑迭代更新，最终得到嵌入向量。相似性得分用两个方法的embedding的余弦距离来表示。
- 神经网络的使用：
    - CCLEARNER：DNN。输入为有八个相似性得分的向量。输出为Clone和NONClone两种标签。
    - White：RNN。输入为每个term的embedding矩阵与句法特征组合得到的repr属性。输出也是克隆和非克隆，并不能区别克隆类型。
    - Gemini：全连接神经网络。输入为 attributed control flow graph, ACFG ，输出为embedding network。
- 评估：
    - CCLEARNER：SourcererCC，NiCad 和 Deckard
    - White：Deckard
    - Gemini：BGM和Genius
- 效果：
    - CCLEARNER：在三型和四型克隆上效果不佳。因为CCLEARNER依赖于在不同方法中使用的完全相同的术语来计算相似性向量。当两个克隆方法共享很少的标识符并包含显着不同的程序结构时，CCLEARNER 无法检测克隆关系。
    - White：（这个是一篇论文里提到的 Guo C, Huang D, Dong N, et al. Deep Review Sharing[C]//2019 IEEE 26th International Conference on Software Analysis, Evolution and Reengineering (SANER). IEEE, 2019: 61-72. ）召回率和f1分数方面检测性能比较差。
    - Gemini：Gemini在大图和小图上都优于BGM和Genius。在相似性检测精度、嵌入生成时间和整体训练时间等方面优先于Genius。


