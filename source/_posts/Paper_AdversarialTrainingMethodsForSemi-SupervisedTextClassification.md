---

title: 【论文精读】Adversarial Training Methods for Semi-Supervised Text Classification

date: 2018.10.27

categories: 
 - [AI testing, 文本对抗样本]

tags: 

 - Deep Learning
 - 论文精读
 - LSTM

description: 能够将对抗训练从有监督学习扩展到半监督学习的虚拟对抗训练
mathjax: true

---
[https://arxiv.org/pdf/1605.07725.pdf](https://arxiv.org/pdf/1605.07725.pdf)
# 基于对抗训练的文本分类的半监督学习方法 论文梳理
摘要：

对抗训练提供了一种规范有监督学习算法的方法，虚拟对抗训练能够将监督学习算法扩展到半监督。
> semi-supervised learning：可以处理部分标记的训练数据，通常是大量未标记数据和少量的标记数据。

然而，这两种方法都需要对输入矢量的多个条目进行小的扰动，但对于稀疏的高维输入（如one-hot）是不合适的。

> one-hot representation：一种最简单的词向量方式是one-hotrepresentation,就是用一个很长的向量来表示一一个词，向量的长度为词典的大小，向量的分量只有一个1，其他全为0，1的位置对应该词在词典中的位置。
例如：“话筒”表示为[0001000000000000...]、“麦克”表示为[0000000010000000..]。每个词都是茫茫0海中的一个1。这种One-hot Representation如果采用稀疏方式存储，会是非常的简洁：也就是给每个词分配一个数字ID。 比如刚才的例子中，话筒记为3，麦克记为8 (假设从0开始记)。如果要编程实现的话，用Hash表给每个词分配一个编号即可。
对某句话进行编码时，将里面的每个单词转换成字典里面这个单词编号对应的位置为1的one-hot矩阵就可以了。
这种词表示有两个缺点: 
(1)容易受维数灾难的困扰，尤其是将其用于Deep Learning的一些算法的时候。由于向量长度是根据单词个数来的，如果有新词出现，这个向量还得增加。
(2)不能很好地刻画词与词之间的相似性(“词汇鸿沟”)。one-hot矩阵相当于简单的给每个单词编了个号，但是单词和单词之间的关系则完全体现不出来。

我们通过对递归神经网络中的 Word Embedding 实施扰动，而不是对原始输入本身进行扰动，将对抗训练和虚拟对抗训练扩展到文本域。 
> Word Embedding，词嵌入，将一个单词转换成固定长度的向量表示，从而便于进行数学处理。有两种方法：
> 1. One hot representation
> 2. Distributed representation：每个向量并不像one-hot方法那样都是稀疏的，而是都有具体的值。映射后的两个单词如果在语义上比较相近，那么着两个单词的词向量（单词所在的点与原点连接的直线所在的向量）就离得比较近。这样做的好处就是对于同义词或者时态不同的词，它们的词向量就会很接近，保留了文章的语义。

所提出的方法在多个基准半监督和纯监督任务上实现了最先进的结果。我们提供可视化和分析，显示：Learned Word Embedding在质量上有所提高，并且在训练时模型不太容易过度拟合。
## 一、介绍

### 对抗样本 Adversarial examples
定义：在输入中增加小扰动，以便找到机器学习模型所引起的loss。 (Szegedy et al., 2014;  Goodfellow et al., 2015)
> loss，机器学习的损失：对糟糕预测的惩罚。损失是一个数值，表示对于单个样本而言模型预测的准确程度。如果模型的预测完全准确，则损失为零，否则损失会较大。

即使加入的干扰小到人类无法察觉，现在的模型也基本上无法正确分类对抗性样本。
### 对抗训练 Adversarial training 
定义：通过对模型进行训练，使模型能正确分类未修改的样本和对抗性样本。
对抗训练不仅提高了对抗性样本的鲁棒性，还提高了原始样本的泛化性能。
> 泛化性能，generalization performance：机器学习算法对新鲜样本的适应能力

对抗训练要求在使用监督成本的训练模型时使用label，因为在对抗扰动被设计成最大化的代价函数中，需要label的信息。
> 代价函数，cost function，用来度量预测错误的程度，通常来说，模型越准确，越接近真实，其cost function的值就越小。

因此对抗训练属于监督学习的范畴，不适用于半监督学习。

### 虚拟对抗训练 Virtual Adversarial Training
虚拟对抗训将对抗训练的概念扩展到半监督制度和未标记的样本。这是通过对模型进行正则化来完成的，以便：给出一个样本，该模型将产生与该样本的对抗扰动产生的输出分布相同的输出分布。
虚拟对抗训练为监督和半监督学习任务实现了良好的泛化性能。


> 《DISTRIBUTIONAL SMOOTHING WITH VIRTUAL ADVERSARIAL TRAINING》
1 正则化 --解决--> 过拟合
    训练样本的数量有限时，容易产生过拟合(overfit)。由训练样本的经验分布计算得到的训练误差，必然不同于测试误差。针对过度拟合的一种流行的对策是：向目标函数添加正则化项，使目标函数的最优参数(the optimal parameter)更少地依赖于似然项(the likelihood term)。
2 新型正则化项：局部分布平滑度 --解决--> 促进模型分布的平滑
    现实中的时间序列或图像，往往都是连续的。而我们输入到模型中的数据，往往是不连续的。连续的输入往往能产生较好的模型泛化能力。
    于是我们创造一种新型正则化项 local distributional smoothness 局部分布平滑度（LDS），它给每个输入数据点周围的输入，奖励模型分布的平滑性，来促进模型分布的平滑。作者将基于LDS的正则化命名为虚拟对抗训练(VAT)。 
    传统对抗训练，用“加入噪声的原始数据输入到模型中的输出概率”与label来计算分布差异；而LDS中将label替换为“原始数据输入到模型中的输出概率”，与“加入噪声的原始数据输入到模型中的输出概率”进行分布差异计算。

### 思路
1. 处理高维离散输入：在continuous word embeddings上进行扰动
对抗性扰动通常包括对很多实值输入进行小的修改。对于文本分类，输入是离散的，通常表示为一系列高维 one-hot 矢量。因为高维 one-hot 矢量集不允许无穷小扰动，所以我们定义了在连续单词嵌入(continuous word embeddings)上的扰动，而不是在离散字输入(discrete word inputs)上的扰动。

2. 只用来通过稳定分类函数来规范文本分类器
传统的对抗性和虚拟对抗性训练，既可以被解释为正规化策略 regularization strategy，也可以防御能够提供恶意输入的对手。由于扰动嵌入不会映射到任何单词，并且对手可能无法访问单词嵌入层(word embedding layer)，因此我们提出的训练策略不再用于防御对手。因此，我们提出这种方法仅作为通过稳定分类函数来规范文本分类器的手段。

3. 文本分类适用于半监督学习
我们采用Dai＆Le（2015）提出的无监督训练的神经语言模型方法，实现了多个半监督文本分类任务的最先进性能，包括情感分类和主题分类。我们仅优化一个额外的超参数ǫ，即限制对抗扰动大小的范数约束，实现了这种最先进的性能。
这些结果强烈鼓励将我们提出的方法用于其他文本分类任务。我们认为文本分类是半监督学习的理想设置，因为有大量未标记的语料库可供半监督学习算法使用。这项工作是我们所知道的第一项使用对抗性和虚拟对抗性训练来改进文本或RNN模型的工作。

4. 定性分析训练的效果
我们还分析了训练有素的模型，以定性地描述对抗性和虚拟对抗性训练的效果。我们发现，对抗训练和虚拟对抗训练改进了基础方法上的word embeddings。


## 二、模型
### RNN 
> 参考：
https://blog.csdn.net/heyongluoyao8/article/details/48636251 

RNN 用来处理序列数据。在传统的神经网络模型中，是从输入层到隐含层再到输出层，层与层之间是全连接的，每层之间的节点是无连接的。但是这种普通的神经网络对于很多问题却无能无力。例如，你要预测句子的下一个单词是什么，一般需要用到前面的单词，因为一个句子中前后单词并不是独立的。

RNNs之所以称为循环神经网路，即一个序列当前的输出与前面的输出也有关。具体的表现形式为网络会对前面的信息进行记忆并应用于当前输出的计算中，即隐藏层之间的节点不再无连接而是有连接的，并且隐藏层的输入不仅包括输入层的输出还包括上一时刻隐藏层的输出。理论上，RNNs能够对任何长度的序列数据进行处理。但是在实践中，为了降低复杂性往往假设当前的状态只与前面的几个状态相关，下图便是一个典型的RNNs： 

![IMAGE](/images/Paper_AdversarialTrainingMethodsForSemi-SupervisedTextClassification/8047B2EAC7AEA9D1203E558BED8D9418.jpg)

RNNs包含输入单元(Input units)，输入集标记为$\{x_0,x_1,...,x_t,x_{t+1},...\}$，而输出单元(Output units)的输出集则被标记为$\{y_0,y_1,...,y_t,y_{t+1},...\}$。RNNs还包含隐藏单元(Hidden units)，我们将其输出集标记为$\{s_0,s_1,...,s_t,s_{t+1},...\}&，这些隐藏单元完成了最为主要的工作。你会发现，在图中：有一条单向流动的信息流是从输入单元到达隐藏单元的，与此同时另一条单向流动的信息流从隐藏单元到达输出单元。在某些情况下，RNNs会打破后者的限制，引导信息从输出单元返回隐藏单元，这些被称为“Back Projections”，并且隐藏层的输入还包括上一隐藏层的状态，即隐藏层内的节点可以自连也可以互连。
例如，对一个包含5个单词的语句，那么展开的网络便是一个五层的神经网络，每一层代表一个单词。
-  $x_t$表示第$t(t=1,2,3...)$步(time step)的输入。比如，$x_1$为第二个词的one-hot向量(根据上图，$x_0$为第一个词)； 
- $s_t$为隐藏层的第$t$步的状态，它是网络的记忆单元。$s_t$根据当前输入层的输出与上一步隐藏层的状态进行计算。$s_t=f(Ux_t+Ws_{t−1})$，其中$f$一般是非线性的激活函数，如$tanh$或$ReLU$，在计算$s_0$时，即第一个单词的隐藏层状态，需要用到$s_{−1}$，但是其并不存在，在实现中一般置为0向量；
- $o_t$是第$t$步的输出，如下个单词的向量表示，$o_t=softmax(Vs_t$). 

### LSTM
> 参考：
https://blog.csdn.net/gzj_1101/article/details/79376798
https://blog.csdn.net/aliceyangxi1987/article/details/73421052

#### LSTM网络
long short term memory，即我们所称呼的LSTM，是为了解决长期以来问题而专门设计出来的，所有的RNN都具有一种重复神经网络模块的链式形式。在标准RNN中，这个重复的结构模块只有一个非常简单的结构，例如一个tanh层。
![IMAGE](/images/Paper_AdversarialTrainingMethodsForSemi-SupervisedTextClassification/1FD10426ED9D4C6023E802F49613125D.jpg)
LSTM 同样是这样的结构，但是重复的模块拥有一个不同的结构。不同于单一神经网络层，这里是有四个，以一种非常特殊的方式进行交互。
![IMAGE](/images/Paper_AdversarialTrainingMethodsForSemi-SupervisedTextClassification/993370ED97E99D764A8E94EB242B7C18.jpg)
在上面的图例中，每一条黑线传输着一整个向量，从一个节点的输出到其他节点的输入。粉色的圈代表 pointwise 的操作，诸如向量的和，而黄色的矩阵就是学习到的神经网络层。合在一起的线表示向量的连接，分开的线表示内容被复制，然后分发到不同的位置。
![IMAGE](/images/Paper_AdversarialTrainingMethodsForSemi-SupervisedTextClassification/542CE137FA10A0AC4FDAF6BC3C4455F1.jpg)

#### LSTM核心思想
LSTM的关键在于细胞的状态整个(绿色的图表示的是一个cell)，和穿过细胞的那条水平线。
![IMAGE](/images/Paper_AdversarialTrainingMethodsForSemi-SupervisedTextClassification/992EA7E8C18B047146C3E71120B4677D.jpg)
细胞状态类似于传送带。直接在整个链上运行，只有一些少量的线性交互。信息在上面流传保持不变会很容易。
若只有上面的那条水平线是没办法实现添加或者删除信息的。而是通过一种叫做门（gates） 的结构来实现的。门可以实现选择性地让信息通过，主要是通过一个 sigmoid 的神经层和一个逐点相乘的操作来实现的。
sigmoid 层输出（是一个向量）的每个元素都是一个在 0 和 1 之间的实数，表示让对应信息通过的权重（或者占比）。比如， 0 表示“不让任何信息通过”， 1 表示“让所有信息通过”。

#### 输入门、遗忘门和输出门
1 遗忘门
第一步是通过遗忘门来决定从细胞状态中丢弃什么信息，即决定了上一个时刻的细胞状态 $C_{t−1}$有多少保留到当前时刻 $C_{t}$。该门会读取$h_{t−1}$和$x_t$，输出一个在0到 1之间的数值给每个在细胞状态 $C_{t−1}$ 中的数字。1 表示“完全保留”，0 表示“完全舍弃”。
![IMAGE](/images/Paper_AdversarialTrainingMethodsForSemi-SupervisedTextClassification/DA9A9D93917F17DB8549928F51E66B80.jpg)
其中$h_{t−1}$表示的是上一个cell的输出，$x_t$表示的是当前细胞的输入。$W_f$ 是遗忘门的权重矩阵，$[h_t-1, x_t] $表示把两个向量连接成一个更长的向量，$b_f$ 是遗忘门的偏置项，$\sigma$表示sigmod函数。

在语言模型中，细胞状态可能包含当前主语的性别，因此正确的代词可以被选择出来。当我们看到新的主语，我们希望忘记旧的主语。

2 输入门
下一步是决定让多少新的信息加入到细胞状态中来，即决定了当前时刻网络的输入$x_t$有多少保留到$c_t$。实现这个需要包括两个步骤：首先，一个叫做“input gate layer ”的 sigmoid 层决定哪些信息需要更新；一个 tanh 层生成一个向量，也就是备选的用来更新的内容，$\tilde{C_t}$ 。在下一步，我们把这两部分联合起来，对 cell 的状态进行一个更新。

![IMAGE](/images/Paper_AdversarialTrainingMethodsForSemi-SupervisedTextClassification/F10FCB1D32F60470CD952C3A4489A69A.jpg)

更新旧细胞状态，$C_{t−1}$更新为$C_t$：
我们把旧状态$C_{t−1}$与遗忘门$f_t$相乘，丢弃掉我们确定需要丢弃的信息。接着加上状态$\tilde{C_t}$ 按元素乘以输入门$i_t$。这就是新的候选值，根据我们决定更新每个状态的程度进行变化。
在语言模型的例子中，这就是我们实际根据前面确定的目标，丢弃旧代词的性别信息并添加新的信息的地方。
![IMAGE](/images/Paper_AdversarialTrainingMethodsForSemi-SupervisedTextClassification/4B3C2C28DBCCC4E8434E6AB67322AAD7.jpg)

3 输出门
最终，我们需要确定输出什么值，即确定细胞状态$C_{t}$有多少输入到LSTM的当前输出值$h_{t}$。首先，我们运行一个 sigmoid 层来确定细胞状态的哪个部分将输出出去。接着，我们把细胞状态通过 tanh 进行处理（得到一个在 -1 到 1 之间的值）并将它和 sigmoid 门的输出相乘，最终我们仅仅会输出我们确定输出的那部分。

在语言模型的例子中，因为他就看到了一个代词，可能需要输出与一个动词相关的信息。例如，可能输出是否代词是单数还是复数，这样如果是动词的话，我们也知道动词需要进行的词形变化。

![IMAGE](/images/Paper_AdversarialTrainingMethodsForSemi-SupervisedTextClassification/EF48C154199890007BE6C4498FC21AC2.jpg)


### 正文
定义一个T个单词的句子为$\{W^{(t)}\mid t=1,...,T\}$，相应的target为$y$。为了将离散的单词转化为连续的向量，我们定义word embedding 矩阵 $\mathbf{V}\in\mathbb{R}^{(K+1) \times D}$，其中 $K$ 是词汇表(vocabulary)中的单词数，每排$v_k$对应第$i$个单词的word embedding。第$(K+1)$个word embedding被用作 end of sequence (eos)  token 的 word embedding，$v_{eos}$。作为一个文本分类模型，我们使用一个简单的基于LSTM的神经网络模型：
![IMAGE](/images/Paper_AdversarialTrainingMethodsForSemi-SupervisedTextClassification/9D844D773E80FC44DB56E7C8E11933B8.jpg)
在时间步t，输入的离散单词为 $w^{(t)}$，对应的 word embedding 是 $v^{(t)}$。

我们还尝试了双向LSTM体系结构，因为这是由当前状态的最先进的方法。为了构造用于文本分类的双向LSTM模型，我们在反序列上向图1中描述的单向LSTM模型添加了一个额外的LSTM。然后，模型预测序列的两端的级联LSTM输出上的标签。

在对抗训练和虚拟对抗训练中，我们训练分类器使之对embedding的扰动具有鲁棒性，如图1b所示。这些扰动在第3节中有详细描述。目前，理解扰动是有界范数(bounded norm)就足够了。该模型可以通过学习具有非常大的范数的嵌入来学习使扰动无关紧要。
为了防止这种病态解决方案，当我们对上面定义的模型应用对抗和虚拟对抗训练时，我们用标准化嵌入$v_k$替换嵌入$\bar{v}_k$，
定义为：
$$ \bar{v}_k = \frac{v_k  - E(v)}{\sqrt{Var(v)}} where E(v) = \sum_{K }^{j = 1}f_jv_j,Var(v)=f_j(v_j-E(v))^2$$

其中，$v_k$是词向量，$\overline{v_k}$是归一化之后的词向量。$f_i$是第i个词的频率，用所有训练样本计算出来的。


##三、对抗训练和虚拟对抗训练

> $ p(y \mid  x + r_{adv};\theta)$ 指给定$x+r_{adv}$，$\theta$的条件下得到y的概率。


对抗训练是一种用于分类器的新颖的正则化方法，用于提高对小、近似最坏情况扰动的鲁棒性。让我们把$x$作为输入，把$\theta$作为分类器的参数。当应用于分类器时，对抗训练将下列术语添加到成本函数：
$$ -log p(y \mid  x + r_{adv};\theta) where r_{adv} = arg min _{r, \left \| r \right \| \leq \varepsilon }log p(y \mid  x + r;\hat{\theta})$$ 

> $r_{adv}$：使后面这个式子达到最小值时的 r 的取值。
每一步中，求得一个最坏的扰动$r_{adv}$去对抗当前的模型，使得预测的概率最小（达到扰动的效果）、对分类结果产生最大的偏差。 而训练时，整体的cost函数最大化预测概率（模型越偏离实际，其cost function的值就越大）。
找到使cost function最大的$r_{adv}$后，再通过调节$\theta$ 的方式使该代价函数尽可能的小，即，使该模型能够不受该扰动的影响（训练模型对扰动的鲁棒性）

其中，$r$是对输入的干扰，$\hat{\theta }$是分类器的当前参数的常数集。使用常数拷贝$\hat{\theta}$而不是$\theta$表明反向传播算法不应用于在对抗样本构造过程中传播梯度。在训练的每个步骤中，我们识别等式2上的当前模型$p(y \mid  x ;\hat{\theta})$的最坏情况扰动$r_{adv}$。相对于$\theta$通过最小化等式2来训练模型对这种扰动的鲁棒性。
但是，通常我们不能精确地计算这个值，因为对于许多有趣的模型如神经网络，关于$r$的精确最小化是困难的。Goodfellow 等人提出了通过线性化$x$上的$log p(y \mid  x ;\hat{\theta})$来近似这个值的方法。在等式2中用线性逼近和L2范数约束，得到的对抗扰动是：
$$ r_{adv} = -\varepsilon g/\left \|g  \right \|_2 where g = \bigtriangledown _x log p(y \mid  x ;\hat{\theta})$$
这个扰动可以容易地用神经网络反向传播来计算。

虚拟对抗训练是一种与对抗训练密切相关的正规化方法。
> 虚拟对抗训练,就是对输入数据加上一个微小的干扰，而这个微小的干扰需要满足一个约束，即原始的模型分布和加入干扰后的模型分布要尽量一致，更通俗地说，就是原始的模型的输出要和加入干扰后的模型输出尽量一致。LDS正则项就是原始模型分布和干扰模型分布之间的KL散度，将LDS正则项加入损失函数中，就是虚拟对抗训练。
原始的对抗训练和虚拟对抗训练很相似，同样都是加入干扰，但是满足的约束不一样。原始的对抗训练要求加入干扰后的模型预测正确的类标的概率要尽量大，这和虚拟对抗网络还是有一点区别的。

虚拟对抗训练引入的额外成本如下：

$$KL[p(\cdot \mid x ; \hat{\theta}) \parallel p(\cdot \mid x+r_{v-adv} ; \theta)]$$
$$ where r_{v-adv} = arg max _{r, \left \| r \right \| \leq \varepsilon }KL[p(\cdot \mid x ; \hat{\theta}) \parallel p(\cdot \mid x+r ; \hat{\theta})]$$

> KL散度( KL divergence)
全称：Kullback-Leibler Divergence 
用途：比较两个概率分布的接近程度 


其中$KL[p \parallel q]$表示分布p和q之间的KL散度。通过最小化等式3，分类器被训练为平滑的。这可以被认为是使分类器抵抗其在当前模型$p(y \mid  x ;\hat{\theta})$上最敏感的方向上的扰动 。 虚拟对抗性损失等式3仅需要输入$x$并且不需要实际标签$y$，而等式2中定义的对抗性损失需要标签$y$。这使得将虚拟对抗训练应用于半监督学习成为可能。虽然我们一般也无法分析计算虚拟对抗性损失，但Miyato等人提出用反向传播有效地计算近似的等式3。

如第二节所述，在我们的工作中，我们将对抗扰动应用于word embeddings单词嵌入，而不是直接应用于输入。 为了定义 word embeddings 的对抗性扰动，让我们将（标准化的）单词嵌入向量序列$[\bar{v}^{(1)},\bar{v}^{(2)},...,\bar{v}^{(T)}]$连接为$s$，将给定$s$的$y$的模型条件概率表示为$p(y \mid  s ;\theta)$，其中θ是模型参数。
然后我们将$s$上的对抗扰动$r_{adv}$定义为：
$$  r_{adv} = -\varepsilon g/\left \|g  \right \|_2 where g = \bigtriangledown _s log p(y \mid  s ;\hat{\theta}) $$
为了使等式5中定义的对抗扰动具有鲁棒性，我们通过以下方法来定义对抗性损失：
$$L_{adv}(\theta) = -\frac{1}{N} \sum_{n=1}^{N}log p (y_n \mid  s_n + r_{adv,n};\theta) $$

其中$N$是标记示例的数量。 在我们的实验中，对抗训练指的是使用随机梯度下降最小化负对数似然加$L_{adv}$。
在我们的文本分类模型的虚拟对抗训练中，在每个训练步骤中，我们计算下面近似的虚拟对抗扰动：
$$r_{v-adv} = \varepsilon g/\left \|g  \right \|_2 where g = \bigtriangledown _{s+d} KL[p(\cdot \mid s ; \hat{\theta}) \parallel p(\cdot \mid s+d ; \hat{\theta})]$$

其中$d$是TD维小随机向量。 该近似对应于二阶泰勒展开和等式3中的幂方法的单次迭代，如先前的工作（Miyato等人，2016）。 然后虚拟对抗性损失定义为：
$$L_{v-adv}(\theta) = -\frac{1}{N'} \sum_{n'=1}^{N'} KL[p(\cdot \mid s_{n'} ; \hat{\theta}) \parallel p(\cdot \mid s_{n'} + r_{v-adv,n'}; \theta)]$$

其中$N'$是标记和未标记样本的数量。
有关对抗性训练方法的最新综述，请参阅Warde-Farley＆Goodfellow（2016）。
> 虚拟对抗损失被定义为：模型的后验分布与每个输入数据点周围局部扰动的鲁棒性。

##四、实验设置
[Adversarial Text Classification](https://github.com/tensorflow/models/tree/master/research/adversarial_text)


为了将我们的方法与其他文本分类方法进行比较，我们测试了5种不同的文本数据集。
我们总结了表1中每个数据集的信息。

IMDB ：用于情感分类的标准基准电影评论数据集。
Elec ：亚马逊电子产品评论数据集。
Rotten Tomatoes：烂番茄，包括电影评论的短片段，用于情感分类。烂番茄数据集没有单独的测试集，因此我们将所有示例随机分为训练集的90％和测试集的10％。我们用不同的随机种子重复训练和评估五次。对于烂番茄数据集，我们还使用Amazon Reviews数据集中的电影评论收集了未标记的示例。 
DBpedia ：维基百科页面的类别分类数据集。因为DBpedia数据集没有其他未标记的示例，DBpedia上的结果仅用于监督学习。
RCV1：路透社语料库的新闻文章。对于RCV1数据集，我们遵循以前的工作，我们在第二级主题上进行了单一主题分类任务。我们使用相同的划分为训练，测试和未标记的集合作为Johnson＆Zhang。关于预处理，我们将任何标点符号视为空格。
我们在Rotten Tomatoes，DBpedia和RCV1数据集上将所有单词转换为小写。我们删除了所有数据集中仅出现在一个文档中的单词。在RCV1上，我们还删除了Lewis等人提供的英语单词列表中的单词。 

表1:数据集摘要。请注意，没有提供烂番茄数据集的未标记样本，因此我们使用未标记的Amazon review数据集。
![IMAGE](/images/Paper_AdversarialTrainingMethodsForSemi-SupervisedTextClassification/753536B11C36F46B3F26E8D4DBE3BD21.jpg)

###4.1 循环语言模型预训练
> 那么预训练的思想就是，该模型的参数不再是随机初始化，而是先有一个任务进行训练得到一套模型参数，然后用这套参数对模型进行初始化，再进行训练。

在Dai＆Le之后，我们使用预先训练的循环语言模型初始化嵌入矩阵和LSTM权重，该模型对标记和未标记的样本进行了训练。

我们使用了具有1024个隐藏单元的单向单层LSTM。单词嵌入维度D在IMDB上为256，在其他数据集上为512。 我们使用带有1024个候选样本的采样softmax损失进行训练。为了优化，我们使用了Adam optimizer，批量大小为256，初始学习率为0.001，每个训练步骤的学习率为0.9999的指数衰减因子。 我们训练了100,000步。我们在除了word embeddings之外的所有参数上应用了梯度剪裁与规范设置为1.0。为了减少GPU上的运行时间，我们使用截断的反向传播，从序列的每一端开始最多400个字。对于循环语言模型的正则化，我们在具有0.5放弃率的单词嵌入层上应用了丢失(dropout)。

对于双向LSTM模型，我们对标准顺序和逆序序列使用了512个隐藏单元LSTM，并且我们使用了与两个LSTM共享的256维字嵌入。 其他超参数与单向LSTM相同。我们在IMDB，Elec和RCV上测试了双向LSTM模型，因为数据集中有相对较长的句子。

使用循环语言模型进行预训练对于我们测试的所有数据集的分类性能非常有效，因此我们在第5节中的结果与此预训练有关。

> softmax函数：
这个softmax的输入是T\*1的向量，输出也是T\*1的向量（也就是图中的prob[T*1]，这个向量的每个值表示这个样本属于每个类的概率），只不过输出的向量的每个值的大小范围为0到1。

> softmax loss：
基于softmax层的损失函数，其中$L=-\sum\limits_{j=1}^T y_{j}\log{s_{j}}$
首先L是损失。Sj是softmax的输出向量S的第j个值，前面已经介绍过了，表示的是这个样本属于第j个类别的概率。yj前面有个求和符号，j的范围也是1到类别数T，因此y是一个1*T的向量，里面的T个值，而且只有1个值是1，其他T-1个值都是0。那么哪个位置的值是1呢？答案是真实标签对应的位置的那个值是1，其他都是0。所以结果为：$L=-\log{s_{j}}$

> Adam优化器：
对梯度的一阶矩估计（First Moment Estimation，即梯度的均值）和二阶矩估计（Second Moment Estimation，即梯度的未中心化的方差）进行综合考虑，计算出更新步长。

### 4.2 训练分类模型

在预训练之后，我们训练了如图1a所示的文本分类模型，如第3节所述，进行对抗和虚拟对抗训练。在目标y的softmax层和LSTM的最终输出之间，我们添加了一个隐藏层，在IMDB，Elec和烂番茄上有维度30，在DBpedia和RCV1上有128。隐藏层上的激活函数是ReLU。为了优化，我们再次使用了Adam优化器，其初始学习率为0.0005，指数衰减为0.9998。批量大小在IMDB，Elec，RCV1上为64，在DBpedia上为128。对于烂番茄数据集，对于每个步骤，我们采用一批64的大小来计算负对数似然和对抗性训练的损失，以及512用于计算虚拟对抗性训练的损失。也适用于烂番茄，我们在未标记的数据集中使用长度T小于25的文本。我们在除IMDB和DBpedia之外的所有数据集上迭代了10,000个训练步骤，我们分别使用了15,000和20,000个训练步骤。我们再次将除了单词嵌入之外的所有参数的标准剪切为1.0。我们还使用截断的反向传播，最多400个单词，并且还从序列的每一端产生多达400个单词的对抗性和虚拟对抗性扰动。

我们发现双向LSTM收敛速度更慢，因此我们在训练双向LSTM分类模型时迭代了15,000个训练步骤。

对于每个数据集，我们将原始训练集划分为训练集和验证集，并且我们粗略地优化了与所有方法共享的一些超参数;（模型体系结构，batch size，训练步骤）以及具有嵌入dropout的基础模型的验证性能。

对于每种方法，我们使用验证集优化了两个标量超参数。 这些是嵌入的dropout和对抗性和虚拟对抗性训练的规范约束。请注意，对于对抗性和虚拟对抗性训练，我们在应用嵌入dropout后生成扰动，我们发现这是最好的。 我们没有尽早用这些方法停止。 仅使用预训练和嵌入dropout的方法用作基线（在每个表中称为基线）。

## 五、实验结果

### 5.1 IMDB数据集的测试性能和模型分析

图2显示了IMDB测试集的学习曲线，其中包括基线方法（仅嵌入dropout和预训练），对抗训练和虚拟对抗训练。我们可以在图2a中看到，对抗性和虚拟对抗性训练实现了比基线更低的负对数似然。此外，虚拟对抗训练可以利用未标记的数据，保持这种低负对数似然，而其他方法在训练后开始过度拟合。关于图2b和2c中的对抗性和虚拟对抗性损失，我们可以看到与负对数似然相同的趋势; 虚拟对抗训练能够使这些值低于其他方法。因为对抗性训练仅对训练数据的labeled子集进行操作，所以它最终overfit甚至抵抗对抗性扰动的任务。

![IMAGE](/images/Paper_AdversarialTrainingMethodsForSemi-SupervisedTextClassification/C165A5DD2A69833217C0EDD550E27368.jpg)

图2：（a）负对数似然，（b）对抗性损失（在等式（6）中定义）和（c）在IMDB上的虚拟对抗性损失（在等式（8）中定义）的学习曲线。在测试集上评估所有值。用$\varepsilon = 5.0$评估对抗性和虚拟对抗性损失。 $\varepsilon$的最佳值在对抗训练和虚拟对抗训练之间有所不同，但5.0的值对两者都有很好的表现，并且提供了一致的比较点。

表2显示了每种训练方法对IMDB的测试性能。“对抗性+虚拟对抗性”意味着共同使用范数约束$\epsilon$来约束对抗性和虚拟对抗性损失。由于只有嵌入dropout，我们的模型实现了7.39％的错误率。 对抗性和虚拟对抗性训练改善了相对于我们基线的表现，虚拟对抗训练取得了与现有技术水平相当的5.91％错误率。 尽管现有技术模型需要训练双向LSTM，而我们的模型仅使用单向LSTM。 我们还使用双向LSTM显示结果。 我们的双向LSTM模型与具有虚拟对抗训练的单向LSTM具有相同的性能。

一个常见的误解是，对抗性训练相当于对有噪声的例子进行训练。噪声实际上是比对抗性扰动弱得多的正则化因子，因为在高维输入空间中，平均噪声向量近似与成本梯度正交。 明确选择对抗性扰动以不断增加成本。 为了证明对抗训练优于增加噪声的优越性，我们在对序列中的每个嵌入中涵盖对照实验，该对象实验用来自具有缩放范数的多元高斯的随机扰动代替对抗性扰动。 在表2中，“带有标记实例的随机扰动”是我们用随机扰动代替$r_{adv}$的方法，“带有标记和未标记实例的随机扰动”是用随机扰动代替$r_{v-adv}$的方法。 每种对抗训练方法都优于每种随机扰动方法。




为了可视化对抗和虚拟对抗训练对嵌入的影响，我们检查了使用每种方法训练的嵌入。表3显示了具有经过训练的嵌入的“good”和“bad”的10个最接近的邻居。由于语言模型预训练步骤，基线和随机方法都受语言的语法结构的强烈影响，但不受文本分类任务的语义的强烈影响。例如，“bad”出现在基线方法的“good”的最近邻居列表和随机扰动方法中。 “bad”和“good”都是可以修改同一组名词的形容词，因此语言模型为它们分配类似的嵌入是合理的，但这显然没有传达关于单词实际含义的大量信息。对抗训练确保句子的意义不能通过一个小的变化来反转，所以这些具有相似语法角色但意义不同的词语会分开。当使用对抗性和虚拟对抗性训练时，“bad”不再出现在“good”的10个最近邻居中。对于虚拟对抗训练，“bad”落到对抗训练的第19个最近邻居和虚拟对抗训练的第21个最近邻居，余弦距离分别为0.463和0.464。对于基线和随机扰动方法，余弦距离分别为0.361和0.377。在另一个方向上，’bad’的最近邻居包括’good’作为基线方法和随机扰动方法的第4个最近邻居。对于两种对抗方法，’good’会掉到’bad’的第36个最近邻居。

我们还调查了15个最近邻居的“great”及其与训练嵌入的余弦距离。 我们看到对抗和虚拟对抗训练（0.159-0.331）的余弦距离比基线和随机扰动法（0.244-0.399）小得多。

表2：IMDB情绪分类任务的测试性能。 *表示使用预训练的CNN嵌入和双向LSTM。
![IMAGE](/images/Paper_AdversarialTrainingMethodsForSemi-SupervisedTextClassification/772F3B8840E087E729E1F3D1380EE665.jpg)


表3：使用每种方法训练的嵌入词一词，“好”和“坏”的10个最接近的邻居。 我们使用余弦距离作为指标。 “基线”表示使用嵌入dropout进行训练，“随机”表示使用标记样本进行随机扰动训练。 “对抗性”和“虚拟对抗性”意味着对抗性训练和虚拟对抗性训练。

![IMAGE](/images/Paper_AdversarialTrainingMethodsForSemi-SupervisedTextClassification/6CC9E24D1F7087D041349FBF300638BC.jpg)
具有经过训练的嵌入的“good”和“bad”的10个最接近的邻居

在虚拟对抗训练之后，弱得多的正面词“好”也从第3个最近的邻居移动到第15个。

> 确保嵌入中的小扰动不会改变情绪分类结果
由于语言模型预训练步骤，基线和随机方法都受语言的语法结构的强烈影响，但不受语义的强烈影响。
所以“bad”出现在 “good”的最近邻居列表的基线和随机扰动方法中。显然，没有传达关于单词实际含义的信息。
对抗训练能确保句子的意义不能通过一个小的变化来反转，所以这些具有相似语法结果但意义不同的词语会分开。
当使用对抗性和虚拟对抗性训练时，“bad”不再出现在“good”的10个最近邻居中。
对于虚拟对抗训练，“bad”落到对抗训练的第19个最近邻居和虚拟对抗训练的第21个最近邻居。



### 5.2 ELEC，RCV1和烂番茄数据集的测试性能
表4显示了Elec和RCV1数据集的测试性能。 我们可以看到我们提出的方法改进了基线方法的测试性能，并在两个数据集上实现了最先进的性能，即使现有技术方法使用CNN和双向LSTM模型的组合。我们的单向LSTM模型改进了现有技术的方法，我们的双向LSTM方法进一步改善了RCV1的结果。 双向模型在RCV1数据集上具有更好性能的原因在于，在RCV1数据集上，与其他数据集相比，存在一些非常长的句子，并且双向模型可以更好地处理具有较短依赖性从逆序句的长句子。

表4：Elec和RCV1分类任务的测试性能。 *表示使用预训练的CNN嵌入，†表示使用预训练的CNN嵌入和双向LSTM。
![IMAGE](/images/Paper_AdversarialTrainingMethodsForSemi-SupervisedTextClassification/7B0DEEF2B63C94899AB36CC504434E3A.jpg)



表5显示了烂番茄数据集的测试性能。 对抗训练能够在基线方法上得到改善，并且在对抗和虚拟对抗成本的情况下，实现了与当前最先进方法几乎相同的性能。 然而，仅虚拟对抗训练的测试表现比基线差。 我们推测这是因为烂番茄数据集的标签句子很少，标注的句子非常短。

在这种情况下，未标记示例上的虚拟对抗性损失超过了监督损失，因此模型优先考虑对扰动具有鲁棒性而不是获得正确答案。

表5：烂番茄情绪分类任务的测试表现。 *表示使用来自word2vec Google News的预训练嵌入，†表示使用来自亚马逊评论的未标记数据。

![IMAGE](/images/Paper_AdversarialTrainingMethodsForSemi-SupervisedTextClassification/93BC33A4F134A773C7CFDC2B19E00BA3.jpg)

### 5.3 对DBPEDIA的性能进行了严格的监督分类任务
表6显示了DBpedia上每种方法的测试性能。 “随机扰动”与第5.1节中解释的“带标记示例的随机扰动”的方法相同。
请注意，DBpedia只有标记了的示例，正如我们在第4节中所解释的那样，因此这项任务是纯粹的监督学习。 我们可以看到基线方法已经几乎达到了当前的最新技术水平，并且我们提出的方法从基线方法改进。
![IMAGE](/images/Paper_AdversarialTrainingMethodsForSemi-SupervisedTextClassification/2B5093620FA000A5B1540289343BAD28.jpg)
## 六、相关工作
Dropout是一种广泛用于包括文本在内的许多领域的正则化方法。之前的一些工作是在训练期间向输入和隐藏层添加随机噪声，以防止过度拟合。然而，在我们的实验和以前的工作中（Miyato等，2016），对抗性和虚拟对抗性扰动的训练优于具有随机扰动的方法。
对于使用神经网络的半监督学习，一种常见的方法，特别是在图像领域，是训练生成模型，其潜在特征可以用作分类的特征。这些模型现在在图像域上实现了最先进的性能。然而，这些方法需要许多具有生成模型的额外超参数，并且生成模型将提供良好的监督学习性能的条件知之甚少。相比之下，对抗性和虚拟对抗性训练仅需要一个超参数，并且具有作为稳健优化的直接解释。

对抗性和虚拟对抗性训练类似于一些半监督或转导性SVM方法，因为两种方法都使得决策边界远离训练实例（或者在转导SVM的情况下，测试实例）。然而，对抗性训练方法坚持输入空间的边距，而SVM坚持由内核函数定义的特征空间上的边距。该属性允许对抗训练方法在对边缘施加的空间上实现具有更灵活功能的模型。在我们的实验中（表2,4）和Miyato等人。 （2016），对抗性和虚拟对抗性训练比基于SVM的方法获得更好的性能。

还有一些半监督方法应用于CNN和RNN的文本分类。这些方法利用“视图嵌入”（Johnson＆Zhang，2015b; 2016b），它使用围绕单词的窗口来生成其嵌入。当这些被用作分类模型的预训练模型时，发现它们可以提高泛化性能。这些方法和我们的方法是互补的，因为我们表明我们的方法从复发的预训练语言模型改进。
## 七、结论
在我们的实验中，我们发现对抗性和虚拟对抗性训练在文本分类任务的序列模型中具有良好的正则化性能。 在所有数据集上，我们提出的方法超出或与现有技术水平相当。 我们还发现，对抗性和虚拟对抗性训练不仅提高了分类性能，还提高了字嵌入的质量。 这些结果表明，我们提出的方法有望用于其他文本域任务，例如机器翻译（Sutskever等，2014），学习单词或段落的分布式表示（Mikolov等，2013; Le＆Mikolov，2014）和 问题回答任务。 我们的方法也可用于其他常规顺序任务，例如视频或语音。

# 问题
> https://github.com/dennybritz/deeplearning-papernotes/blob/master/notes/adversarial-text-classification.md 

- 一致性
有时虚拟对抗训练获胜，有时对抗性+虚拟对抗性获胜。有时，双LSTM会使事情变得更糟，有时更好。背后的故事是什么？我们真的需要尝试所有组合来找出适用于给定数据集的内容吗？
- 缺少一些双LSTM实验
总是让我感到双LSTM实验“因某种原因失踪”
- 数据集之间的超参数和批量大小存在很多差异。 
我想知道为什么。这是否与他们比较的模型保持一致？这些参数是否在验证集上进行了优化（作者说只有dropout和epsilon被优化）？
- 如果对抗性训练比随机排列更强大的正则化，我想知道我们是否仍然需要在嵌入中dropout。不应该采用对抗训练来解决这个问题吗？

# 跑代码
> https://www.cnblogs.com/tensorflownews/p/7298646.html
> https://www.jianshu.com/p/2ea7a0632239

## python3
`brew install python3`
## TensorFlow的安装
先通过命令安装pip
`sudo curl https://bootstrap.pypa.io/ez_setup.py -o - | sudo python`
`sudo easy_install pip`
`sudo easy_install --upgrade pip`更新pip
安装CPU版本：`pip3 install tensorflow`会提示一个当先版本匹配不到的错误提示
```
pip3 install tensorflow 
Collecting tensorflow
Could not find a version that satisfies the requirement tensorflow (from versions: ) No matching distribution found for tensorflow
```
所以要用以下命令：
`pip3 --default-timeout=10000 install --upgrade https://storage.googleapis.com/tensorflow/mac/cpu/tensorflow-1.3.0-py3-none-any.whl`

##Anaconda的安装
[https://www.anaconda.com/download/#macos](https://www.anaconda.com/download/#macos)

## 使用 Anaconda 安装TensorFlow
[https://www.tensorflow.org/install/install_mac#installing_with_anaconda](https://www.tensorflow.org/install/install_mac#installing_with_anaconda)
1.执行以下命令创建名为 tensorflow 的 conda 环境:
`$ conda create -n tensorflow`
2.执行以下命令激活 conda 环境：
```
$ source activate tensorflow
 (tensorflow)$  # Your prompt should change
```
可以看到：
![IMAGE](/images/Paper_AdversarialTrainingMethodsForSemi-SupervisedTextClassification/DD3185EAEBDEE7D4596ABF3EC149EA1B.jpg)
3.验证安装
![IMAGE](/images/Paper_AdversarialTrainingMethodsForSemi-SupervisedTextClassification/6138282E7ABA5F256B6511637CBE632E.jpg)
输出结果中的b是bytes的意思，python3的语法。加‘b’表示按位字符串，‘r’原始意义（常用在带有转义字符的路径中，表示不转义）; ‘u’加字符串表示unicode编码字符串，同时适用于中英文。

4.停用环境：
`source deactivate`

5.如果要卸载 TensorFlow，执行下面的命令
`$ pip3 uninstall tensorflow `
6.删除环境`conda remove -n tensorflow`
# 心酸运行过程
[https://github.com/tensorflow/models/tree/master/research/adversarial_text](https://github.com/tensorflow/models/tree/master/research/adversarial_text)

安装wget
`brew install wget`

## 端到端 imdb 情绪分类
### 获取数据 Fetch data
```
$ wget http://ai.stanford.edu/~amaas/data/sentiment/aclImdb_v1.tar.gz \
    -O /tmp/imdb.tar.gz
$ tar -xf /tmp/imdb.tar.gz -C /tmp
```
目录`/tmp/aclImdb`是原始imdb 数据:
![IMAGE](/images/Paper_AdversarialTrainingMethodsForSemi-SupervisedTextClassification/E15CB7DB331D268633E7BEDBD5CA28E9.jpg)

### 生成词汇 Generate vocabulary
把adversarial_text文件夹专门放一个地方
![IMAGE](/images/Paper_AdversarialTrainingMethodsForSemi-SupervisedTextClassification/8BE905E4BA6590B91818C45F6F07D50B.jpg)

```
$ source activate tensorflow
$ export IMDB_DATA_DIR=/tmp/imdb
$ python gen_vocab.py \
    --output_dir=$IMDB_DATA_DIR \
    --dataset=imdb \
    --imdb_input_dir=/tmp/aclImdb \
    --lowercase=False
```
词汇和频率文件将在 `$IMDB_DATA_DIR`中生成
![IMAGE](/images/Paper_AdversarialTrainingMethodsForSemi-SupervisedTextClassification/E5BB42A0D3F945C66502768D7463186C.jpg)
解决AttributeError: 'collections.defaultdict' object has no attribute 'iteritems'方法：
打开 gen_vocab.py
将第82行`vocab_freqs = dict((term, freq) for term, freq in vocab_freqs.iteritems()`
改成：`vocab_freqs = dict((term, freq) for term, freq in vocab_freqs.items()`

![IMAGE](/images/Paper_AdversarialTrainingMethodsForSemi-SupervisedTextClassification/6C62538F71EC638C16F32685E048B63C.jpg)

现在`/tmp/imdb`文件夹里有:
![IMAGE](/images/Paper_AdversarialTrainingMethodsForSemi-SupervisedTextClassification/A8F98D22D393D89E173C187B5AB9C585.jpg)

## 生成训练、验证和测试数据 Generate training, validation, and test data
```
$ python gen_data.py \
    --output_dir=$IMDB_DATA_DIR \
    --dataset=imdb \
    --imdb_input_dir=/tmp/aclImdb \
    --lowercase=False \
    --label_gain=False
```
![IMAGE](/images/Paper_AdversarialTrainingMethodsForSemi-SupervisedTextClassification/C34ED84961A7A019D595F41E3490F14A.jpg)
`$IMDB_DATA_DIR` 包含TFRecords文件。
![IMAGE](/images/Paper_AdversarialTrainingMethodsForSemi-SupervisedTextClassification/6942FFF5B82833E2DEDD564CE309E4F5.jpg)
## 预训练 imdb 语言模型

```
$ export PRETRAIN_DIR=/tmp/models/imdb_pretrain
$ python pretrain.py \
    --train_dir=$PRETRAIN_DIR \
    --data_dir=$IMDB_DATA_DIR \
    --vocab_size=86934 \
    --embedding_dims=256 \
    --rnn_cell_size=1024 \
    --num_candidate_samples=1024 \
    --batch_size=256 \
    --learning_rate=0.001 \
    --learning_rate_decay_factor=0.9999 \
    --max_steps=100000 \
    --max_grad_norm=1.0 \
    --num_timesteps=400 \
    --keep_prob_emb=0.5 \
    --normalize_embeddings
```

![IMAGE](/images/Paper_AdversarialTrainingMethodsForSemi-SupervisedTextClassification/2A6077E1292265E9F3C27D25DE30D5ED.jpg)
把`--vocab_size=86934 \`改成`--vocab_size=87007 \`
```
(tensorflow) VikingdeMacBook-Pro:adversarial_text viking$ python pretrain.py \
>     --train_dir=$PRETRAIN_DIR \
>     --data_dir=$IMDB_DATA_DIR \
>     --vocab_size=87007 \
>     --embedding_dims=256 \
>     --rnn_cell_size=1024 \
>     --num_candidate_samples=1024 \
>     --batch_size=256 \
>     --learning_rate=0.001 \
>     --learning_rate_decay_factor=0.9999 \
>     --max_steps=100000 \
>     --max_grad_norm=1.0 \
>     --num_timesteps=400 \
>     --keep_prob_emb=0.5 \
>     --normalize_embeddings
INFO:tensorflow:Constructing TFRecordReader from files: ['/tmp/imdb/train_lm.tfrecords']
WARNING:tensorflow:From /Users/viking/Documents/paper/2adversarial/VirtualAdversarialTraining/Adversarial_Text_Classification/adversarial_text/inputs.py:171: string_input_producer (from tensorflow.python.training.input) is deprecated and will be removed in a future version.
Instructions for updating:
Queue-based input pipelines have been replaced by `tf.data`. Use `tf.data.Dataset.from_tensor_slices(string_tensor).shuffle(tf.shape(input_tensor, out_type=tf.int64)[0]).repeat(num_epochs)`. If `shuffle=False`, omit the `.shuffle(...)`.
WARNING:tensorflow:From /Applications/anaconda3/envs/tensorflow/lib/python3.6/site-packages/tensorflow/python/training/input.py:276: input_producer (from tensorflow.python.training.input) is deprecated and will be removed in a future version.
Instructions for updating:
Queue-based input pipelines have been replaced by `tf.data`. Use `tf.data.Dataset.from_tensor_slices(input_tensor).shuffle(tf.shape(input_tensor, out_type=tf.int64)[0]).repeat(num_epochs)`. If `shuffle=False`, omit the `.shuffle(...)`.
WARNING:tensorflow:From /Applications/anaconda3/envs/tensorflow/lib/python3.6/site-packages/tensorflow/python/training/input.py:188: limit_epochs (from tensorflow.python.training.input) is deprecated and will be removed in a future version.
Instructions for updating:
Queue-based input pipelines have been replaced by `tf.data`. Use `tf.data.Dataset.from_tensors(tensor).repeat(num_epochs)`.
WARNING:tensorflow:From /Applications/anaconda3/envs/tensorflow/lib/python3.6/site-packages/tensorflow/python/training/input.py:197: QueueRunner.__init__ (from tensorflow.python.training.queue_runner_impl) is deprecated and will be removed in a future version.
Instructions for updating:
To construct input pipelines, use the `tf.data` module.
WARNING:tensorflow:From /Applications/anaconda3/envs/tensorflow/lib/python3.6/site-packages/tensorflow/python/training/input.py:197: add_queue_runner (from tensorflow.python.training.queue_runner_impl) is deprecated and will be removed in a future version.
Instructions for updating:
To construct input pipelines, use the `tf.data` module.
WARNING:tensorflow:From /Users/viking/Documents/paper/2adversarial/VirtualAdversarialTraining/Adversarial_Text_Classification/adversarial_text/inputs.py:172: TFRecordReader.__init__ (from tensorflow.python.ops.io_ops) is deprecated and will be removed in a future version.
Instructions for updating:
Queue-based input pipelines have been replaced by `tf.data`. Use `tf.data.TFRecordDataset`.
Traceback (most recent call last):
  File "pretrain.py", line 46, in <module>
    tf.app.run()
  File "/Applications/anaconda3/envs/tensorflow/lib/python3.6/site-packages/tensorflow/python/platform/app.py", line 125, in run
    _sys.exit(main(argv))
  File "pretrain.py", line 41, in main
    train_op, loss, global_step = model.language_model_training()
  File "/Users/viking/Documents/paper/2adversarial/VirtualAdversarialTraining/Adversarial_Text_Classification/adversarial_text/graphs.py", line 162, in language_model_training
    loss = self.language_model_graph()
  File "/Users/viking/Documents/paper/2adversarial/VirtualAdversarialTraining/Adversarial_Text_Classification/adversarial_text/graphs.py", line 222, in language_model_graph
    return self._lm_loss(inputs, compute_loss=compute_loss)
  File "/Users/viking/Documents/paper/2adversarial/VirtualAdversarialTraining/Adversarial_Text_Classification/adversarial_text/graphs.py", line 231, in _lm_loss
    embedded = self.layers['embedding'](inputs.tokens)
  File "/Applications/anaconda3/envs/tensorflow/lib/python3.6/site-packages/tensorflow/python/keras/engine/base_layer.py", line 746, in __call__
    self.build(input_shapes)
  File "/Users/viking/Documents/paper/2adversarial/VirtualAdversarialTraining/Adversarial_Text_Classification/adversarial_text/layers.py", line 70, in build
    name='embedding')
  File "/Applications/anaconda3/envs/tensorflow/lib/python3.6/site-packages/tensorflow/python/keras/engine/base_layer.py", line 609, in add_weight
    aggregation=aggregation)
  File "/Applications/anaconda3/envs/tensorflow/lib/python3.6/site-packages/tensorflow/python/training/checkpointable/base.py", line 639, in _add_variable_with_custom_getter
    **kwargs_for_getter)
  File "/Applications/anaconda3/envs/tensorflow/lib/python3.6/site-packages/tensorflow/python/keras/engine/base_layer.py", line 1977, in make_variable
    aggregation=aggregation)
  File "/Applications/anaconda3/envs/tensorflow/lib/python3.6/site-packages/tensorflow/python/ops/variables.py", line 183, in __call__
    return cls._variable_v1_call(*args, **kwargs)
  File "/Applications/anaconda3/envs/tensorflow/lib/python3.6/site-packages/tensorflow/python/ops/variables.py", line 146, in _variable_v1_call
    aggregation=aggregation)
  File "/Applications/anaconda3/envs/tensorflow/lib/python3.6/site-packages/tensorflow/python/ops/variables.py", line 125, in <lambda>
    previous_getter = lambda **kwargs: default_variable_creator(None, **kwargs)
  File "/Applications/anaconda3/envs/tensorflow/lib/python3.6/site-packages/tensorflow/python/ops/variable_scope.py", line 2437, in default_variable_creator
    import_scope=import_scope)
  File "/Applications/anaconda3/envs/tensorflow/lib/python3.6/site-packages/tensorflow/python/ops/variables.py", line 187, in __call__
    return super(VariableMetaclass, cls).__call__(*args, **kwargs)
  File "/Applications/anaconda3/envs/tensorflow/lib/python3.6/site-packages/tensorflow/python/ops/resource_variable_ops.py", line 297, in __init__
    constraint=constraint)
  File "/Applications/anaconda3/envs/tensorflow/lib/python3.6/site-packages/tensorflow/python/ops/resource_variable_ops.py", line 409, in _init_from_args
    initial_value() if init_from_fn else initial_value,
  File "/Applications/anaconda3/envs/tensorflow/lib/python3.6/site-packages/tensorflow/python/keras/engine/base_layer.py", line 1959, in <lambda>
    shape, dtype=dtype, partition_info=partition_info)
  File "/Applications/anaconda3/envs/tensorflow/lib/python3.6/site-packages/tensorflow/python/ops/init_ops.py", line 255, in __call__
    shape, self.minval, self.maxval, dtype, seed=self.seed)
  File "/Applications/anaconda3/envs/tensorflow/lib/python3.6/site-packages/tensorflow/python/ops/random_ops.py", line 236, in random_uniform
    minval = ops.convert_to_tensor(minval, dtype=dtype, name="min")
  File "/Applications/anaconda3/envs/tensorflow/lib/python3.6/site-packages/tensorflow/python/framework/ops.py", line 1050, in convert_to_tensor
    as_ref=False)
  File "/Applications/anaconda3/envs/tensorflow/lib/python3.6/site-packages/tensorflow/python/framework/ops.py", line 1146, in internal_convert_to_tensor
    ret = conversion_func(value, dtype=dtype, name=name, as_ref=as_ref)
  File "/Applications/anaconda3/envs/tensorflow/lib/python3.6/site-packages/tensorflow/python/framework/constant_op.py", line 229, in _constant_tensor_conversion_function
    return constant(v, dtype=dtype, name=name)
  File "/Applications/anaconda3/envs/tensorflow/lib/python3.6/site-packages/tensorflow/python/framework/constant_op.py", line 208, in constant
    value, dtype=dtype, shape=shape, verify_shape=verify_shape))
  File "/Applications/anaconda3/envs/tensorflow/lib/python3.6/site-packages/tensorflow/python/framework/tensor_util.py", line 442, in make_tensor_proto
    _AssertCompatible(values, dtype)
  File "/Applications/anaconda3/envs/tensorflow/lib/python3.6/site-packages/tensorflow/python/framework/tensor_util.py", line 353, in _AssertCompatible
    (dtype.name, repr(mismatch), type(mismatch).__name__))
TypeError: Expected int64, got -1.0 of type 'float' instead.
```
好像是要下载tensorflow 新的版本比如 1.8的版本
`pip3 --default-timeout=10000 install --upgrade https://storage.googleapis.com/tensorflow/mac/cpu/tensorflow-1.8.0-py3-none-any.whl`
还是这个问题...试一试python2.7的版本:
> 创建一个虚拟的环境`conda create -n python27 python=2.7`
激活环境`source activate python27`
安装tenserflow`pip --default-timeout=10000 install --upgrade  https://download.tensorflow.google.cn/mac/cpu/tensorflow-1.8.0-py2-none-any.whl`
退出环境`source deactivate`
删除环境`conda remove -n python27 --all`

然后重新做一遍：
1.生成词汇 没有报错
![IMAGE](/images/Paper_AdversarialTrainingMethodsForSemi-SupervisedTextClassification/A23B5381604C92FB693FC4F78D98C1CA.jpg)
2.生成训练、验证和测试数据
![IMAGE](/images/Paper_AdversarialTrainingMethodsForSemi-SupervisedTextClassification/ED0D3AB151128F21440F60744DB1575E.jpg)
3.预训练 imdb 语言模型 没有报错
但是特别慢啊 就在实验室的gpu服务器跑去吧 再见👋


`$PRETRAIN_DIR`包括预训练的语言模型的检查点。
##训练分类器 Train classifier
大多数flag保持不变，除了删除候选采样(candidate sampling)和添加`pretrained_model_dir`，分类器将从中加载预训练embedding和LSTM变量，以及与对抗训练和分类相关的flag。
```
$ export TRAIN_DIR=/tmp/models/imdb_classify
$ python train_classifier.py \
    --train_dir=$TRAIN_DIR \
    --pretrained_model_dir=$PRETRAIN_DIR \
    --data_dir=$IMDB_DATA_DIR \
    --vocab_size=86934 \
    --embedding_dims=256 \
    --rnn_cell_size=1024 \
    --cl_num_layers=1 \
    --cl_hidden_size=30 \
    --batch_size=64 \
    --learning_rate=0.0005 \
    --learning_rate_decay_factor=0.9998 \
    --max_steps=15000 \
    --max_grad_norm=1.0 \
    --num_timesteps=400 \
    --keep_prob_emb=0.5 \
    --normalize_embeddings \
    --adv_training_method=vat \
    --perturb_norm_length=5.0
```
##评估测试数据  Evaluate on test data
```
$ export EVAL_DIR=/tmp/models/imdb_eval
$ python evaluate.py \
    --eval_dir=$EVAL_DIR \
    --checkpoint_dir=$TRAIN_DIR \
    --eval_data=test \
    --run_once \
    --num_examples=25000 \
    --data_dir=$IMDB_DATA_DIR \
    --vocab_size=86934 \
    --embedding_dims=256 \
    --rnn_cell_size=1024 \
    --batch_size=256 \
    --num_timesteps=400 \
    --normalize_embeddings
```
在GPU上跑出来的结果：
![IMAGE](/images/Paper_AdversarialTrainingMethodsForSemi-SupervisedTextClassification/CB8054359B22DFB4B799B0B25C276417.png)
接下来要改变flag得到其他准确率。
## 代码概述
主要入口点是下面列出的二进制文件。每个训练二进制文件构建一个VatxtModel，在graphs.py中定义，后者又使用inputs.py中定义的图形构建块（定义输入数据读取和解析），layer.py
（定义核心模型组件）和adversarial_losses.py（定义对抗训练损失）。 训练循环本身在train_utils.py中定义。

### 二进制文件
预训练:pretrain.py。
分类器培训:train_classifier.py
评价:evaluate.py。
### 命令行标志Flags
与分布式训练和训练循环本身有关的标志是在train_utils.py定义的。
与模型超参数相关的标志在graphs.py定义。
与对抗训练有关的标志定义在 adversarial_losses.py。
每个任务特有的标志是在主二进制文件中定义的。

### 数据生成
词汇生成: gen_vocab.py
数据生成: gen_data.py 
在document_generators.py中定义的命令行标志控制处理的数据集以及处理的方式。






# 运行环境
OS Platform and Distribution:macOS High Sierra 10.13.6
TensorFlow installed from (source or binary): installed through pip
TensorFlow version (use command below): 1.18.0
Python version: 2.7
