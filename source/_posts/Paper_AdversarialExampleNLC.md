---

title: 【论文精读】Adversarial Example For Natural Language Classification Problems

date: 2018.9.28

categories: 
 - [AI testing, 文本对抗样本]

tags: 

 - Deep Learning
 - 论文精读


description: 通过同义词替换生成对抗样本
mathjax: true

---
# 论文梳理
Anonymous authors. Paper under double-blind review. Under review as a conference paper at ICLR 2018. 
## 一、介绍
### 对抗样本 adversarial example
#### 出处
Christian Szegedy等人在2014年发表的文章：_Intriguing properties of neural networks_
在数据集中通过故意添加细微的干扰所形成的输入样本，导致模型以高置信度给出一个错误的输出。
![IMAGE](/images/Paper_AdversarialExampleNLC/6CFC125A08797BF761A8578F6ACE39CA.jpg)
如上样本x的label为熊猫，在对x添加部分干扰后，在人眼中仍然分为熊猫，但对深度模型，却将其错分为长臂猿，且给出了高达99.3%的置信度。
#### 原因简述
以 $y = W^T \times  X$ 举例($W$是权重，$X$是输入)。如果 $X’ = X + t$，$t$ 为干扰，$W^T \times  X’ = W^T \times  X + W^T \times  t$，也就是多出一个 $W^T \times  t$ 项，$W$ 和 $t$ 维数很大时，即使很小扰动，累加起来也很可观。

### 自然语言分类中的对抗干扰 adversarial perturbations
实验分类：垃圾邮件分类、情感分析、虚假新闻检测
主要思想：同义词替换，而不改变句子原义
如下图所示
![IMAGE](/images/Paper_AdversarialExampleNLC/7A28C1E9EFD1F6938A92887DEE5C730F.jpg)
### 发现
1. 机器学习存在漏洞
2. 分类算法有局限性

##二、背景
分类器是映射 $f:\chi \rightarrow  \Upsilon $ ，其中输入 $x \in \chi$，标签 $y \in \Upsilon$，且标签来自K个类，即 $\Upsilon = \left \{ y_{1}, y_{2},...,y_{K} \right \}$。
分类器 $f$ 是个深度神经网络或者线性模型，它会为输入 $x$ 在每个类 $y_{k}$ 中的可能性做出评分值 $f_{y_{k}}(x)$，并将评分最高的那个类作为输出。

### 图片分类器

对正常输入 $x$ 做扰动后得到 $x'$，将其送入分类器，得到错误的类别 $y'$，这一过程可以如下定能够如下定义：$$f(x')=y' and  \left \| x-x' \right \|\leq \varepsilon $$

优化为：
$$ max_{x'}J(x')    s.t.    \left \| x-x' \right \|\leq \varepsilon $$
其中 $J(x')$ 即为评分值 $f_{y_{k}}(x')$
### 自然语言分类
具有n-gram特性的线性分类器往往具有非常好的性能。循环神经网络RNN与卷积神经网络CNN在文本分类方面都有良好效果。
###自然语言分类与图形分类的区别
1. 自然语言是离散的，输入的x为字符或者单词。
2. 自然语言维度更高，与词汇量成比例。
3. 自然语言比图片像素具有更高的层次，因为原始的单词比原始的像素带有更多信息。

##三、自然语言分类器的对抗样本

产生文本分类器中的对抗样本有两个难点：
1. 话语之间没有度量的简单概念(难以定义一个无法察觉的“扰动”)。
2. 离散输入不适合基于梯度的方法，因此需要新的优化算法。


###  Altered Adversarial Examples
对于一个分类器f，如果存在一个x'使得：
$$f(x')=y' and  \left \| x-x' \right \|\leq \gamma  $$

就说 $x'$ 是一个对抗改动（adversarial alteration）。

其中函数 $c$ 是一种约束 $c:\chi  \times  \chi \rightarrow \mathbb{R}_{+ }^{L}$ ，$\gamma $是一个边界向量（a vector of bounds），且 $ \gamma \in \mathbb{R}^{L} $。 $c$ 和 $\gamma $ 通过约束 $L \leq 1$ 捕捉“不可察觉的改变”的概念。 

### Adversarial Examples for Natural Language Classification

在自然语言环境中，我们希望改动的样本 $x'$ 能与原样本 $x$ 保存相同的意思。为此我们提出了一种特殊的限制函数 $c(x-x')$，使两种表述有相同的含义并保持相同的句法特征（syntactic properties），例如，写作风格应该保持相似。具体来说，函数c由两个约束组成，分别在两个层次上捕捉句子的相似性。

#### 语义层次（semantic similarity）

用思考向量（thought vector）（参考《Efficient estimation of word representations in vector space》）的概念来捕获话语的含义。思考向量可以看作是从句子到向量空间的映射，其中具有相似含义的句子彼此接近。在此意义下，约束被定义为：
$$ \left \| v-v' \right \|_{2}  \leq \gamma_{1}  $$
其中 $v$ 和 $v'$ 是与 $x$ 和 $x'$ 对应的思考向量，$ \gamma_{1}$ 是超参数。
本文中将思考向量设为单个词的向量的平均值。


#### 句法层次（syntactic similarity）

通常thought vector不能捕捉到句法的正确性。比如将对句子中的单词重新排序，可以得到相同的平均词向量。为此，加上句法约束
$$ \left | log P(x')  -  log P(x) \right | <  \gamma _{2} $$
它依赖于语言模型 $ P : \chi \rightarrow \left[0, 1\right ]$ ，我们要保证扰动和原始示例之间的语言模型概率相似（如果$x$  是一个不合语法的句子，即使用不正确的英语，那么 $x_{1}$ 应该保持相似的正确性水平）。

### Greedy Construction of Altered Adversarial Examples
优化
$$ max_{x'}J(x')    s.t.   c(x-x') \leq \gamma $$

其中，$ J(x') $ 就是 $f_{y'}(x')$ 且 $y' \neq  y$
Algorithm 1: Greedy Optimization Strategy for Finding Adversarial Examples
```C
Data: Datapoint x, termination threshold τ , neighborhood size N, parameters γ1, γ2, δ.
We initialize the algorithm at the uncorrupted data point: x' ← x ;
while Objective is below the threshold J(x') < τ and fraction of words replaced is less than δ do
    Create a working set W = ∅ ;
    for each word w in x do
        for each word w¯ among the N closest to w and different from w do
            substitute w' with w¯ to get x¯ and if x¯ satisfies ∣logP(x′)−logP(x)∣<γ2 , then W ← W ∪ {x'};
    Choose highest scoring world replacement  x' ← arg max x¯∈W J(x¯) or if W = ∅, then break ;
return x';
```


#### 思路
对句子中某一单词找 $N$ 个近义词来替换，并且满足句法层次一致，就把新的句子加入 $W$ 集合里。对每个单词都进行这样的操作，最后返回评分最高的句子。

#### 算法分析
算法输入（algorithm inputs）

需要一个目标分类器 $f$，算法1通过优化目标 $J$ 将 $x$ 转化为 $x'$。假设 $x$ 是一组 $n$ 个离散符号（即单词,word），用 $w_{i}$  表示，$i=1,2,...,n$。此算法可以扩展到相似的通常的离散问题。

优化策略（optimization strategies）

首先我们定义一个边界向量 $\delta$，例如使其满足 $\sum_{i=1}^{n}\mathbb{I}\{w_{i}\neq w_{i}'\}\leq \delta \cdot n$ ，这可以使我们排除掉一些显然不满足条件的对抗改动。或者，设置一个关于目标的最小阈值 $\gamma $，例如目标标签分数的最小期望值，在我们达到这个最小阈值时停止算法。

单词替换（word repalcement）

我们用一个合适的词向量空间中的最近邻居（nearest neighbors）替换单词，并考虑N个最近的邻居。这些最近邻居单词很可能出现在相同的文本环境中。为了确保更换的也是同义词,我们使用的GloVE词向量（GloVE word vectors）进行后期处理（出自《 Counter-fitting word vectors to linguistic constraints》）。这确保了向量满足已知的同义词关系所施加的语言约束，并确保具有相似含义的单词在向量空间中彼此接近。
![IMAGE](/images/Paper_AdversarialExampleNLC/B52E3CF25D30702E9DA3C9BF3D50345F.jpg)

##四、实验

### 任务
在垃圾邮件分类(Spam filtering)、情感分析(Sentiment analysis)、虚假新闻检测(Fake news detection)三种自然语言分类任务中使用对抗样本。
![IMAGE](/images/Paper_AdversarialExampleNLC/FFDC46C33F31F4F0D19A6B37C3DDB42F.jpg)
将训练集中的10%用作验证集，所有对抗样本通过测试集产生、评估。同样我们为每一项任务，在训练集中训练一个卦语言模型（trigram language model）。并且实例化 中的词向量语义约束。
### 模型
####朴素贝叶斯
naive Bayes：将每个文档转换为一个词袋（bag-of words）表示形式，然后按照Wang & Manning(2012)的方法，将单词特征进行二值化，并使用一个多项式模型进行分类。
####长短记忆网路
long short-term memory：建立了一个拥有512个隐藏神经元的单层 LSTM 。在输入LSTM前先进行 word2vec 词嵌入，变成300维向量。然后，我们对LSTM在每个时间步上的输出进行平均，得到一个特征向量，用于最后的逻辑回归预测情感。
####浅单词级卷积网络
shallow word-level convolutional networks：用嵌入层(如在LSTM中)训练一个CNN，一个时间卷积层，然后是最大池化，最后用一个全连接层进行分类。
####深字符集卷积网络
deep character-level convolutional networks：
###主要实验
我们手动选择了优化设置，从而在对抗样本的强度和一致性之间做出了合理的权衡。
在所有实验中，我们令阈值 $\tau =0.7$，临近大小 $N=15$，参数 $\gamma_{1}=0.2$ 且 $ \delta_{1}=0.5$。在情感分析和假消息检测中令 $\gamma_{2}=2$，在垃圾信息检测中令 $\gamma_{2}=\infty $。我们还比较了用随机抽样代替算法1中的$arg max$得到的随机扰动。发现模型对于随机扰动的抵抗能力很强。

![IMAGE](/images/Paper_AdversarialExampleNLC/D0E73A3410EEFFD090FFEB03753841A6.jpg)
### 人类评估

通过Amazon Mechanical Turk上的人体实验验证了我们的示例的质量和一致性。

首先，对100个随机测试集示例进行二次抽样，并让人类评估人员为原始数据点及其对抗性更改版本分配标签（例如正面或负面评论）。我们对每个查询平均了五种不同评估的意见。我们发现人类评估者在两种类型的投入上都达到了类似的准确度，这表明我们的对抗性改变足以保证关键语义能够得到人类的认可。人的准确性通常低于算法的准确性：虚假新闻任务本质上是困难的，而非垃圾邮件通常被错误分类，因为“ham”电子邮件没有标准定义;在情绪分析中，两种准确度都在合理的误差范围内。

接下来，我们要求人类注释者按照1到5的等级对同一组示例的“书写质量”进行评级，其中五个最高质量的（可能由人类生成）；一个质量最低的（可能是通过机器生成的）。表5显示人类倾向于为两组样本分配相似的分数。尽管我们的对抗性示例并未完美形成，但这些结果表明它们的质量与原始示例相当（其中还包含多个拼写和语法错误）。
![IMAGE](/images/Paper_AdversarialExampleNLC/015820C10EECDFCF356B15D2F443B38E.jpg)

### 错误分析
我们发现我们的对抗性示例表现出三种错误：句法，语义和事实。
####句法错误
句法错误是不合语法的单词替换，包括
1. 将“isis claim responsibility for shooting（isis声称对射击负责）”替换为“isis petition responsibility for shooting（isis请愿射击的责任）”
这个错误是由于多个单词含义。claim和petition都有请求的意思。

2. 将“never before has an fbi director”替换为“never until has an fbi director”
这个错误是由于单词不相关（并且在单词向量空间中很远）。


####语义错误
当句子的含义被改变时，就会出现语义错误。

1. 大多数情况下，这是由于多个词的意义相同。例如，“isis claim responsibility for shooting”到“isis claim responsibility for filming” 
2. 词语嵌入错误。例如，“iisis claim responsibility for ceasefire”。


####事实错误
当句子变得明显错误时，事实错误是一种特殊情况。当“三月十六日星期一”改为“三月十六日星期四”或“联邦调查局副局长詹姆斯卡尔斯特罗姆”改为“五角大楼助理导演詹姆斯卡尔斯特罗姆”，或“共和党人支持特朗普”改为“支持奥巴马的共和党人”时。
这些可能不是假评论或假新闻的问题，并且可以通过专门技术来补救，例如，通过performing Named Entity Recognition （执行命名实体识别）。

### 传递性
图像分类模型的一个有趣的特性是，为一个分类器生成的对抗性示例可能被其他分类器错误分类。 我们还检查了对抗文本是否在四个模型之间转移，重点关注Yelp数据集。 如表6所示，模型之间存在中等程度的可转移性。

![IMAGE](/images/Paper_AdversarialExampleNLC/A1C1673A497E4F0AEA85E4A2EEC60273.jpg)

三个单词级别模型（NB，LSTM，WCNN）的对抗性示例也没有对字符级别深度CNN和其他单词级别模型进行推广，这表明输入表示（字符或单词）的选择是一个因素，这会影响可转移性。

#探究
## 统计语言模型（Statistical Language Model）
> 参考 https://www.cnblogs.com/f-young/p/7906451.html


语言模型就是用来计算一个句子的概率的模型，即P(W1,W2,...Wk)。利用语言模型，可以确定哪个词序列的可能性更大，或者给定若干个词，可以预测下一个最可能出现的词语。举个音字转换的例子来说，输入拼音串为nixianzaiganshenme，对应的输出可以有多种形式，如你现在干什么、你西安再赶什么、等等，那么到底哪个才是正确的转换结果呢，利用语言模型，我们知道前者的概率大于后者，因此转换成前者在多数情况下比较合理。再举一个机器翻译的例子，给定一个汉语句子为李明正在家里看电视，可以翻译为Li Ming is watching TV at home、Li Ming at home is watching TV、等等，同样根据语言模型，我们知道前者的概率大于后者，所以翻译成前者比较合理。

### 计算一个句子的概率
给定句子（词语序列）S=W1,W2,...,Wk，它的概率可以表示为：
$$P(S)=P(W_{1},W_{2},W_{3},...,W_{k}) =p(W_{1})p(W_{2}|W_{1})P(W_{3}|W_{1}W_{2})...p(W_{k}|W_{1},W_{2},...,W_{k}) $$
由于上式中的参数过多，因此需要近似的计算方法。常见的方法有n-gram模型方法、决策树方法、最大熵模型方法、最大熵马尔科夫模型方法、条件随机域方法、神经网络方法，等等。
### n-gram语言模型
n-gram模型也称为n-1阶马尔科夫模型，它有一个有限历史假设：当前词的出现概率仅仅与前面n-1个词相关。
$$ P(S) =P(W_{1},W_{2},W_{3},...,W_{k}) \approx  \prod_{i=1}^{k}P(W_{i}|W_{1},...,W_{i-1}) $$
当n取1、2、3时，n-gram模型分别称为unigram、bigram和trigram语言模型。n-gram模型的参数就是条件概率 $P(W_{i}|W_{i-n+1},...,W_{i-1})$ 。假设词表的大小为100,000，那么n-gram模型的参数数量为$100,000^{n}$。n越大，模型越准确，也越复杂，需要的计算量越大。最常用的是bigram，其次是unigram和trigram，n取≥4的情况较少。

假设上面的n不取很长，而只取2个，那么就可以大大减少计算量。此时，假设一个词$w_{i}$出现的概率只与它前面的$w_{i−1}$有关，这种假设称为1阶马尔科夫假设。

$$P(w_1,w_2,...,w_n) \approx P(w_1)P(w_2\vert w_1)P(w_3\vert w_2)...P(w_n\vert w_{n-1})$$

那么，接下来的问题就变成了估计条件概率 $P(w_{i}\vert w_{i−1})$，根据它的定义 $P(w_i\vert w_{i-1}) = \frac{P(w_i,w_{i-1})}{P(w_{i-1})}$
当样本量很大的时候，基于大数定律，一个短语或者词语出现的概率可以用其频率来表示，即
$$P(w_i,w_{i-1}) \approx \frac{count(w_i,w_{i-1})}{count(*)}$$
$$P(w_{i-1}) \approx \frac{count(w_{i-1})}{count(*)}$$
其中，count(i)表示词i出现的次数，count表示语料库的大小。
那么
$$ P(w_i\vert w_{i-1}) = \frac{P(w_i,w_{i-1})}{P(w_{i-1})} \approx \frac{count(w_i,w_{i-1})}{count(w_{i-1})}$$
### 高阶语言模型
考虑以下两个问题，对于二元模型：
- 如果此时 $count(w_i,w_{i-1})=0$，是否可以说 $P(w_i\vert w_{i-1})=0$ ?
- 如果此时 $count(w_i,w_{i-1})=count(w_{i-1})$，是否可以说 $P(w_i\vert w_{i-1})=1$ ?

显然不能。
古德图灵估计的原理：
>对于没有看见的事件，我们不能认为他发生的概率就是0，因此从概率的总量中，分配一个很小的比例给这些没有看见的事件。这样一来，看见的那些事件的概率就要小于1了，因此，需要将所有看见的事件的概率调小一点。至于小多少，要根据“越是不可信的统计折扣越多”的方法进行。

### 神经网络语言模型
n-gram模型不足
- 没有考虑上下文中更多的词提供的信息
- 没有考虑词与此之间的相似性


神经网络语言模型可以概括为以下三点：
- 将词汇表中的每个词表示成一个在m维空间里的实数形式的分布式特征向量
- 使用序列中词语的分布式特征向量来表示连接概率函数
- 同时学习特征向量和概率函数的参数

特征向量表示词的不同特征：
每一个词都是向量空间内的一个点。特征的个数通常都比较小，比如30，60或者100，远远小于词汇表的长度。概率函数是在给定一个词前面的若干词的情况下，该词出现的条件概率。调整概率函数的参数，使得训练集的对数似然达到最大。每个词的特征向量是通过训练得到的，也可以用先验知识进行初始化。

## 词向量
参考
>https://blog.csdn.net/mawenqi0729/article/details/80698350

### 词的两种表示方式
one-hot representation 和 distribution representation。

1. 离散表示
one-hot representation把每个词表示为一个长向量。这个向量的维度是词表大小，向量中只有一个维度的值为1，其余维度为0，这个维度就代表了当前的词。 
例如： 
苹果 [0，0，0，1，0，0，0，0，0，……] 
one-hot representation相当于给每个词分配一个id，这就导致这种表示方式不能展示词与词之间的关系。另外，one-hot representation将会导致特征空间非常大，但也带来一个好处，就是在高维空间中，很多应用任务线性可分。
2. 分布式表示
word embedding指的是将词转化成一种分布式表示，又称词向量。分布 
式表示将词表示成一个定长的连续的稠密向量。

分布式表示优点: 
(1)词之间存在相似关系： 
是词之间存在“距离”概念，这对很多自然语言处理的任务非常有帮助。 
(2)包含更多信息： 
词向量能够包含更多信息，并且每一维都有特定的含义。在采用one-hot特征时，可以对特征向量进行删减，词向量则不能。


### 生成词向量的方法
生成词向量的方法方法都依照一个思想：任一词的含义可以用它的周边词来表示。
生成词向量的方式可分为：基于统计的方法和基于语言模型(language model)的方法。