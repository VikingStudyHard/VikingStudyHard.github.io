---

title: 【论文精读】A Novel Neural Source Code Representation based on Abstract Syntax Tree
date: 2019.7.31

categories: 
 - [代码克隆]

tags: 
 - Code Clone
 - 论文精读

description: 代码表示，AST-based Neural Network
mathjax: true

---
> Zhang J, Wang X, Zhang H, et al. A novel neural source code representation based on abstract syntax tree[C]//Proceedings of the 41st International Conference on Software Engineering. IEEE Press, 2019: 783-794.


# 做什么
represent source code，可适用于不可编译和不完整的代码片段。
先前方法的不足：
- 传统方法将程序视为自然语言文本进行分析并基于token建模，如果只是简单地用基于text或基于token的方法处理，会忽略重要的语义、句法信息。
- 基于神经模型的抽象语法树（abstract syntax tree, AST）能更有效地表示源代码，但一方面AST又大又深时，会在训练中出现**梯度消失**，并且难以捕捉远程依赖，这将遗漏远程节点从根节点携带的一些语义，例如叶节点中的标识符；另一方面将AST转化为完整的二叉树会**破坏掉原有的句法结构**，并且AST更深了，这样会使神经模型捕获更多真实和复杂语义的能力下降，使长期依赖性问题更严重。一种解决方案是引入显式（长期）控制流和数据依赖图（control flow and data flow dependencies），并采用图嵌入（Graph Embedding）技术来表示源代码。但这种程序依赖图（procedural program dependency graphs，PDGs）不适用于不可编译和不完整的代码片段，因此妨碍了对任意代码片段的应用。

基于AST的神经网络（AST-based Neural Network，ASTNN），比当下最先进的基于AST的神经网络学到更多的词法、句法信息。

# 怎么做
## 简介
{% img https://vikingstudyhard.github.io/images/Paper_ASTNN/32D0AD535D03BF21C1C5EFA5A6CEF3D9.png 1464 %}
基于AST的神经网络将一个代码片段的大AST分解成一系列小的**statement tree**中，在所有语句树上执行基于树的神经嵌入。通过捕获词法、statement级的句法信息将语句树转化为**vector**。这里statement指的是在程序语言规范中定义的Statement AST节点。并将 MethodDeclaration 视为特殊语句节点。基于一系列语句向量，使用**双向RNN**将用于**平衡语句的自然性**并最终产生**代码片段的向量**表示。

例如：
{% img https://vikingstudyhard.github.io/images/Paper_ASTNN/C55FA35603C51B3C0085EAEFABA1406E.jpg 983 %}

首先，我们从代码片段构建一个AST，并将整个AST拆分为小语句树（一个树由一个语句的AST节点组成，并以Statement节点为根）。例如，在图1中，语句树用虚线表示，原始代码片段中的相应语句（或statement headers）也用虚线标记。
其次，我们在多路语句树上设计一个RNN编码器来捕获语句级的词法和句法信息，然后在语句向量中表示它们。
第三，基于语句向量序列，我们使用双向门控递归单元（GRU），利用语句的顺序自然性，最终获得整个向量表示代码片段。

## 具体方法
ASTNN的总体结构：
{% img https://vikingstudyhard.github.io/images/Paper_ASTNN/5B403822B3A5CCC55EAE2E4152D338AC.jpg 608 %}
### 解析AST并构建ST树
将源代码片段解析为AST，并设计一个前序遍历算法，将每个AST拆分为一系列语句树（ST树是由语句节点作为根和语句的相应AST节点组成的树），按语句的粒度进行拆分，并通过顺序遍历提取语句树的序列。

给定一个AST $T$和一组语句AST节点$S$，$T$中的每个语句节点$s \in S$对应一个源代码语句。将MethodDeclaration视为特殊语句节点，所以 $S=S \cup\{MethodDeclaration\}$。对于嵌套语句，定义一组独立的节点$P=\{block,body\}$，其中 $block$ 用于分割嵌套语句（例如Try和While语句）的header和body，$body$ 用于方法声明的主体。所有语句节点 $s \in S$ 的后代用 $D(s)$ 定义。对任意$d \in D(s)$，如果存在一条从 $s$ 到 $d$ 通过一个节点 $p \in P $ 的一条路径，则意味着节点 $d$ 被包含在语句 $s$ 的body中的一个语句中。我们将节点 $d$ 称为 $s$ 的一个子节点。 然后，由语句节点 $s \in S$ 为根的语句树（ST树）是由节点 $s$ 及其所有后代组成的树，不包括其在 $T$ 中的子语句节点。

例如，由 $MethodDeclaration$ 生根的第一个ST树被 图1（b）中的虚线包围，包括诸如“static”，“public”和“readText”之类的header部分，并且排除了文中两个$LocalVariable$、一个$Try$和一个$Return$语句的节点。

由于一个ST树的节点可能具有三个或更多个子节点，因此我们也将其称为多路ST树以将其与二叉树区分开。以这种方式，可以将一个大AST分解为一系列非重叠和多路的ST树。

通过遍历器和构造器，AST的分割和ST树序列的构造是直接的。遍历器在预先排序的**深度优**先中通过AST访问每个节点，并且构造器递归地创建ST树，顺序地添加到ST树序列。这种做法保证了按源代码中的顺序附加新的ST树。

通过这种方式得到ST树的序列作为ASTNN的原始输入。

**选择AST的分割粒度**。我们在这项工作中选择ST-tree，因为语句是携带源代码语义的基本单元。如果所选粒度的大小太大（例如，完整的AST）可能会遇到梯度消失问题。但是如果它太小（例如，AST的节点级别），则模型将成为基于令牌的RNN，会捕获更少的语句的语法知识。我们的实验结果表明，提出的语句级粒度更好，因为它在ST树的大小和句法信息的丰富性之间有很好的平衡。
### 在多路ST树上编码语句
#### 语句向量
对于给定的ST树，设计了基于语句编码器的RvNN，用于学习语句的向量表达。

由于AST中有各种特殊的句法符号，我们通过AST的前序遍历（preorder traversal）获得所有符号作为训练语料库。 word2vec 用于学习符号的无监督向量，并且训练的符号的嵌入在语句编码器中用作初始参数。因为表示诸如标识符之类的词汇信息的AST的所有叶节点也被并入ST树的叶节点中，所以我们的符号嵌入可以很好地捕获词汇知识。

{% img https://vikingstudyhard.github.io/images/Paper_ASTNN/EF37285978298FEFD6A04550D95EB429.jpg 478 %}

以图1中$MethodDeclaration$节点为根的第一个ST树为例，编码器遍历ST树，并递归地将当前节点的符号作为计算的新输入，连同其子节点的隐藏状态。如图3所示。我们仅在此处显示前两个级别。在ST树中，两个子节点$readText$（即方法名称）和$FormalParameter$（即定义方法参数的语法结构）以及其他兄弟节点丰富了$MethodDeclara tion$的含义。如果我们将ST树转换为一个二叉树，例如将$readText$的节点移动到$FormalParameter$节点的一个子节点或后代，原始语义可能会被破坏。因此，我们采用原始的多路ST树作为输入。

具体地，给定ST树$t$，令$n$表示非叶节点，$C$表示其子节点的数量。

首先，使用预训练的嵌入参数 $W_{e} \in \mathbb{R}^{|V| \times d}$，其中$V$是词汇量大小，$d$是符号的嵌入维度，节点$n$的词汇向量可以通过以下方式获得：
$$
v_{n}=W_{e}^{\top} x_{n}
$$
其中$x_{n}$是符号$n$的one-hot表示，$v_{n}$是嵌入。

接下来，节点$n$的向量表示通过以下等式计算：
$$
h=\sigma\left(W_{n}^{\top} v_{n}+\sum_{i \in[1, C]} h_{i}+b_{n}\right)
$$

其中$W_{n} \in \mathbb{R}^{d \times k}$是编码维度为$k$的权重矩阵，$b_n$是偏差项，$h_i$是每个孩子的隐藏状态$i$，$h$是更新的隐藏状态，$\sigma$是激活函数，例如 tanh 或 identity function。 我们在本文中使用了 identity function。类似地，我们可以递归地计算和优化ST树中的所有节点的向量。此外，为了确定节点向量的最重要特征，将所有节点推入堆栈，然后通过 max pooling 进行采样。也就是说，我们通过下式得到ST树的最终表示和相应语句，其中N是ST树中的节点数。
$$
e_{t}=\left[\max \left(h_{i 1}\right), \cdots, \max \left(h_{i k}\right)\right], i=1, \cdots, N
$$
这些语句向量能捕获语句的词法和语句级别的句法信息。

#### 批处理
{% img https://vikingstudyhard.github.io/images/Paper_ASTNN/7E1A548A3A223963AA67447E3B97E7D1.jpg 423 %}

为了提高大数据集的训练效率，有必要设计批处理算法以同时编码多个样本（即代码片段）。 然而，通常在多路ST树上进行批处理使得困难，因为子节点的数量在一个批次的相同位置中关于父节点而变化。例如，图4中给定具有3个子节点的两个父节点${ns}_1$和具有2个子节点的${ns}_2$，由于不同的C值，不可能在一个批次中直接计算两个父节点的等式2。 为了解决这个问题，我们设计了一种在算法1中动态处理批量样本的算法。

{% img https://vikingstudyhard.github.io/images/Paper_ASTNN/F02F11236547548F8B0D0570FF52FE57.jpg 477 %}

直观地说，尽管父节点具有不同数量的子节点，但算法可以动态地检测并将具有相同位置的所有可能子节点放入组中，然后通过利用矩阵运算以批量方式加速每组的方程2的计算。 

在算法1中，我们批处理ST树的L个样本，然后从根节点开始广度优先遍历它们（第4行）。对于批次相同位置的当前节点ns，我们首先批量计算等式1（第10行），然后通过节点位置检测并分组所有子节点（第12-16行）。如图4所示，我们将子节点按其位置分成三组，并将这些组记录在数组列表C和C I中。基于这些组，我们递归地对所有子节点执行批处理（第17-21行）。在获得所有子节点的结果之后，我们批量计算等式2（第22行），并将批量ST树的所有节点向量推送到堆栈S（第24行）。最后，我们可以通过公式3（第5行）中描述的合并来获得ST树样本的向量和相应的语句。

#### 代表一系列语句

基于一系列ST树向量，我们利用GRU追踪语句的自然性。
给定一个代码片段，假设有从它的AST中提取的ST树 $T$，并且让 $Q \in \mathbb{R}^{T \times k}=\left[e_{1}, \cdots, e_{t}, \cdots, e_{T}\right], t \in[1, T]$ 表示向量在序列中编码的ST树。在时间t，过渡方程如下：
$$
\begin{aligned} r_{t} &=\sigma\left(W_{r} e_{t}+U_{r} h_{t-1}+b_{r}\right) \\ z_{t} &=\sigma\left(W_{z} e_{t}+U_{z} h_{t-1}+b_{z}\right) \\ \tilde{h}_{t} &=\tanh \left(W_{h} e_{t}+r_{t} \odot\left(U_{h} h_{t-1}\right)+b_{h}\right) \\ h_{t} &=\left(1-z_{t}\right) \odot h_{t-1}+z_{t} \odot \tilde{h}_{t} \end{aligned}
$$
其中$r_t$是用于控制先前状态的影响的复位门，$z_t$是用于组合过去和新信息的更新门，$h$是候选状态并且用于与先前状态$h_t-1$一起进行线性$t$插值以确定现状$h_t$。$h_{t} . W_{r}, W_{z}, W_{h}, U_{r}, U_{z}, U_{h} \in \mathbb{R}^{k \times m}$是权重矩阵，$b_r，b_z，b_h$是偏差项。在迭代地计算所有时间步骤的隐藏状态之后，可以获得这些语句的顺序自然性。

$$
\begin{aligned} \overrightarrow{h_{t}} &=\overrightarrow{GRU}\left(e_{t}\right), t \in[1, T] \\ \overleftarrow h_{t} &=\overleftarrow {GRU}\left(e_{t}\right), t \in[T, 1] \\ h_{t} &=\left[\overrightarrow{h_{t}}, \overleftarrow{h_{t}}\right], t \in[1, T] \end{aligned}
$$

与语句编码器类似，然后通过最大池化或平均池化对这些状态的最重要特征进行采样。 考虑到不同语句的重要性在直觉上是不相等的，例如，MethodInvocation语句中的API调用可能包含更多功能信息，因此我们使用max pooling来捕获默认情况下最重要的语义。该模型最终产生向量$r \in \mathbb{R}^{2 m}$，其被视为代码片段的向量表示。

## 应用
代码克隆检测。在软件工程研究中，检测代码克隆的方法得到了广泛的研究，即检测两个代码片段是否实现相同的功能。假设存在代码片段向量r1和r2，它们的距离通过$r = | r1- r2 |$来测量语义相关性。然后我们可以将输出yˆ = sigmoid（xˆ）∈[0，1]视为它们的相似性，其中xˆ = Wor + bo。损失函数定义为二进制交叉熵：


$$
J(\Theta, \hat{y}, y)=\sum(-(y \cdot \log (\hat{y})+(1-y) \cdot \log (1-\hat{y})))
$$
为了针对这两项任务训练ASTNN模型，目标是最大程度地减少损失。我们在本文中使用AdaMax，因为它计算效率高。
优化所有参数后，将存储经过训练的模型。对于新的代码片段，应将其预处理为ST树序列，然后馈入重新加载的模型中以进行预测。输出是不同标签的概率p。

对于克隆检测，p是[0,1]范围内的单个值，因此我们可以通过以下方式获得预测，其中$\delta$是阈值：
$$
prediction =\left\{\begin{array}{ll}{True , } & {p>\delta} \\ {False,} & {p \leq \delta}\end{array}\right.
$$

## 评估方法

对于克隆检测，我们的方法分别在两个基准数据集上将F1值的结果从82％提高到93.8％和59.4％到95.5％。

### 数据集
一个数据集由从Online Judge（OJ）系统收集并由Mou等人公开的简单C程序组成， OJ基准测试中的程序适用于104个不同的编程问题。如果程序旨在解决相同的问题，则它们具有相同的功能。
另一个数据集BigCloneBench（BCB）由Svajlenko等提供，用于评估代码克隆检测工具。BCB由来自大数据项目间Java存储库的已知正确和错误肯定克隆组成。作为基准，这两个数据集已被许多研究人员用于代码相似和克隆检测。表一总结了与我们两项任务相对应的基本统计数据。

{% img https://vikingstudyhard.github.io/images/Paper_ASTNN/649E9409F48A217FDAABD651B58238AB.jpg 272 %}

### 实验设定

我们使用pycparser和javalang工具获取AST，分别用于C和Java代码。对于这两个任务，我们都使用word2vec 和Skip-gram算法训练了符号的嵌入，并将嵌入大小设置为128。ST树编码器和双向GRU的隐藏尺寸为100。我们将最小批量大小设置为这两个任务分别为64和最多15和5个纪元。克隆检测的阈值设置为0.5。

对于每个数据集，我们将其随机分为三个部分，其中用于训练，验证和测试的比例分别为60％，20％，20％。我们使用学习率为0.002的优化器AdaMax进行训练。所有实验均在具有16个2.4GHz CPU内核和Titan Xp GPU的服务器上进行。



代码克隆检测中通常有四种不同类型的代码克隆。Type-1是相同的代码片段，除了注释和布局不同之外；除了Type-1，Type-2是相同的代码片段，只是标识符名称和文字值不同。 Type-3是语法上相似的代码段，它们在语句级别上有所不同。Type-4是在语法上不同的代码段，它们实现相同的功能。

对于BCB，克隆对的相似性定义为：基于行和基于令牌的度量的平均结果。
- Type-1和Type-2的两个片段的相似度为1。
- Type-3进一步分为强3型和中度3型，其相似性分别在[0.7,1）和[0.5,0.7)。
- Type-4的相似性在[0,0.5）范围内，其克隆对占所有克隆类型的98％以上。

在OJ中，针对同一问题的两个程序形成了一个未知类型的克隆对。
如表1所示，从OJ的前15个编程问题中的每一个中选择500个程序，即OJClone。它会产生超过2800万个克隆对，这对于比较而言非常耗时，因此我们随机选择5万个样本。同样，我们从BCB解析了将近600万个真实克隆对和26万个错误克隆对。

我们将我们的方法与包括RAE和CDLH在内的现有最新神经网络模型进行克隆检测。对于RAE，程序的无监督向量是通过作者的开源工具7获得的，我们将其用于有监督的培训，即RAE +。其配置根据其论文进行设置。

CDLH没有被作者公开，因此我们直接从论文中引用他们的结果，因为他们的实验与我们的实验共享相同的数据集。由于在RAE和CDLH中已经比较了其他传统的克隆检测方法（如DECKARD）和常见的神经模型（如doc2Vec8），因此我们在实验中将其省略。类似于代码分类的实验，我们还与OJClone中的两种基于PDG的图嵌入方法进行了比较。但是，BCB主要包含不完整且无法编译的方法级代码片段，我们无法获取其PDG。

由于可以将代码克隆检测公式化为二进制分类问题（是否克隆），因此我们选择常用的精度（P），召回率（R）和F1度量（F1）作为评估指标。

 


## 实验效果

在这个研究问题中，我们想探究我们的模型对于代码克隆检测这一具有挑战性的问题是否有效。我们对OJClone和BCB数据集进行实验。

从OJClone，我们对5万个克隆对进行采样以进行实验。
在BCB中，我们首先对2万个假克隆对进行采样，作为阴性样本。对于Type-1到强Type-3，我们提取属于该类型的所有真实克隆对作为正样本，因为它们的数目少于2万。对于其他类型，我们采样了2万个真正的克隆对。然后，我们将它们分为五个组以检测每种类型。

详细结果显示在表III（BCB）和IV（OJClone）中。如前所述，在表III中，我们引用了CDLH结果。由于没有报告详细克隆类型的P和R值，因此我们将其替换为“-”。

{% img https://vikingstudyhard.github.io/images/Paper_ASTNN/61F3E43B92FD9344DEDB4CBA7DD35039.jpg 679 %}

BCB-ST3和BCB-MT3分别表示BCB中的强类型3和中类型3，依此类推。
BCB-ALL是根据各种克隆类型的百分比得出的加权总和结果。
在表IV中，我们将前面所述的另外两种基于PDG的图嵌入方法PDG + HOPE和PDG + GGNN进行了比较。


###  BCB 

我们首先讨论RAE+，CDLH和ASTNN模型在BCB上的性能。显然，这三种方法在识别Type-1和Type-2中两个代码片段的相似性方面都非常有效，因为除了不同的标识符名称，注释等之外，这两个代码片段几乎相同。

对于其他类型的BCB，RAE +的性能比其他两种方法差很多，因为它**没有存储历史信息的机制**，例如CDLH和ASTNN中的LSTM或GRU。

将CDLH与我们的方法进行比较，我们可以看到ASTNN在F1度量方面优于CDLH，尤其是对于Type-4。在BCB Type-4中，错误的克隆对也共享语法相似性，这被证实是巧合，并且很难区分。这表明我们的ASTNN模型可以克服第III部分中描述的局限性，并捕获语句的顺序自然性，从而比CDLH捕获更多的语法差异和复杂的语义。


###  OJClone
在OJClone中，RAE+和我们的模型可以观察到相似的结果。
但是，CDLH的性能要比BCB差得多。与BCB不同，OJClone中程序的变量名通常是没有意义的，因此CDLH可能会丢失很多词汇信息，并且只能捕获某些语法信息。

### 自身比较
{% img https://vikingstudyhard.github.io/images/Paper_ASTNN/2D435462CA514659F10D5D2ACC9C3C69.jpg 484 %}



AST-Block 比 AST-Full 和 AST-Node 效果好。

LSTM和GRU在OJClone上差不多，在BCB上GRU效果好一点。（GRU参数少，结构比LSTM简单）


# 问题

将statement vector整合成大vector，输入的embedding序列不能表现出嵌套结构。

