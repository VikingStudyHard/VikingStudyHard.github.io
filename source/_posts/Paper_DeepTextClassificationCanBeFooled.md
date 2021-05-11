---

title: 【论文精读】Deep text classification can be fooled

date: 2019.02.15

categories: 
 - [AI testing, 文本对抗样本的生成]

tags: 

 - Deep Learning
 - 论文精读


description:  TextFool方法
mathjax: true

---

https://arxiv.org/abs/1704.08006

> Liang, B., Li, H., Su, M., Bian, P., Li, X., & Shi, W. (2017). Deep text classification can be fooled. arXiv preprint arXiv:1704.08006.


Liang等人认为有效的文本对抗样本要满足以下三个条件：
1) 欺骗目标DNN；
2) 扰动不可察觉，即精心设计的文本不会吸引人类观察者的注意力；
3) 保持效用，即文本的语义应该保持不变，且人类观察者能毫不费力地正确分类。

例如：广告的垃圾邮件
1) 不能被过滤系统删掉
2) 不应有乱码、病句等
3) 有效地传递广告内容，人类能得到产品信息的正确推广（知道他要卖什么）

**难点：第二点**
文本是一种离散数据，而图像或音频数据是连续的。连续数据在某种程度上容忍扰动；而文本则不然。

即使微小的扰动也会将一个字或一个单词变成一个完全不同的单词和一个单词，以至于无法识别。因此，当直接采用针对文本的现有多媒体数据目标扰动算法时，得到的文本样本可能会失去其原始含义，甚至对于人类观察者而言变得毫无意义。

比如：快速梯度符号方法，简称FGSM

> Ian J Goodfellow, Jonathon Shlens, and Christian Szegedy. Explaining and harnessing adversarial examples. In Proceedings of the 2015 International Conference on Learning Representations. Computational and Biological Learning Society, 2015.



<div align=center>
  <img width=725 src="https://vikingstudyhard.github.io/images/Paper_DeepTextClassificationCanBeFooled/EA562C58C2FBE102761F27F58CC53696.jpg" >
</div>

人类肉眼几乎无法察觉图片中加入的噪声。

如果利用成本梯度来制作文本的对抗样本：

<div align=center>
  <img width=472 src="https://vikingstudyhard.github.io/images/Paper_DeepTextClassificationCanBeFooled/B131D9CFBCE1EE401F96AC8FEACCDB0E.jpg" >
</div>

第一行：原始文本是摩托车的简短描述，通过DNN模型正确地归类为“运输工具”类。

第二行：直接用FGSM生成对抗样本，预测更改为“公司”类，但新文本不可读。

第三行：只对具有最高梯度幅度的几个字符进行操作（一个具有较大梯度的像素对当前的分类结果有更大的贡献）。但生成的文本不自然，有明显的扰动。

这样得到的文本样本可能会失去其原始含义，甚至对于人类而言变得毫无意义。

# 做什么
Liang 等人提出了基于DNN的文本分类器的对抗样本的生成方法，他们设计了三种策略（即插入，修改和删除）并引入自然语言水印技术，用有效的单词、连贯的句子进行改动以减少人类对扰动的察觉，此方法可以将对抗样本分类到指定的类。

# 怎么做

主要思想：使用该类训练数据中的 **单词频率** 来衡量每个单词对某个类的 **重要性**

以字符级DNN为例：

{% label primary@热字符 %} Hot characters：计算每个训练样本$x$的成本梯度$\bigtriangledown _{x}J \left (F,x,c \right)$，将包含具有最大梯度值的维度的字符称为热字符。

{% label primary@热训练短语 %} Hot Training Phrases（HTP）：把目标类中所有训练样本都收集起来，每个样本根据经验选择前50个热字符，通过一个简单的扫描识别出包含三个或三个以上热字符的短语，从中确定最常用的短语，称为热训练短语 —— 来自训练语料库 —— 揭示了要插入的内容

<div align=center>
  <img width=447 src="https://vikingstudyhard.github.io/images/Paper_DeepTextClassificationCanBeFooled/060A810E846522D9523A72A45F2D448C.jpg" >
</div>

Building类十大热训练短语
“历史性的（historic）”是建筑类（building class）中的HTP，它出现了7279次。

{% label primary@热样本短语 %} Hot Sample Phrases（HSP）：对于给定文本样本，仍然使用反向传播算法来定位对当前分类具有显着贡献的热词，并且这些短语被识别为热样本短语 —— 来自句子本身 —— 用来确定在何处操纵

例如：
<div align=center>
  <img width=447 src="https://vikingstudyhard.github.io/images/Paper_DeepTextClassificationCanBeFooled/CB4055DD395EE3048C18233597DE490A.jpg" >
</div>

在“*principal stock exchange of Uganda. It was founded*”之前插入一个HTP（“*historic*”）可以成功地将描述Company的文本错误地分类为具有88.6％置信度的Building类。


设计了三种策略（即插入，修改和删除）并引入自然语言水印技术，可以将对抗样本分类到指定的类。

## 白盒方案
三类策略
### 插入策略
对给定文本中插入一些新的文本项，使文本分到其他类别。如之前的例子。

在实践中，经常需要多次插入。但是，将多个HTP直接插入文本可能会损害其实用性和可读性。

策略： **插入事实**和 **伪造事实**

#### 插入事实
通过搜索互联网或某些事实数据库，得到与插入点密切相关并包含目标类的一些理想的HTPs。

<div align=center>
  <img width=458 src="https://vikingstudyhard.github.io/images/Paper_DeepTextClassificationCanBeFooled/1421DF3D32A877E3BEAACF4C592F72C6.jpg" >
</div>

本来被分为专辑Album的文本（99.9%），由于其中出现了发行该专辑的公司的名字（YG Entertainment），所以在之后插入该公司的介绍，使之被分到Company类（94%），其中 *“company”、“founded”、“entertainment”* 都是Company类的热训练短语。

#### 插入伪造事实

<div align=center>
  <img width=487 src="https://vikingstudyhard.github.io/images/Paper_DeepTextClassificationCanBeFooled/6B3206E3811D20B35CE2E29E8E30FC57.jpg" >
</div>

本来是被分到交通工具类（99.9%），但加了这句伪造事实，就被分类到电影类（90.2%），其中 *“romantic”, “movie”, “directed by”,“American”*都是电影类的热训练短语。

问题：容易通过互联网检索而被发现是假的，所以要找很难证伪的句子来插入


### 修改策略
修改哪个词 —— HSP —— 在给定的句子中哪部分对分类贡献最大。

有两张方式修改HSP：
1.从相关的语料库中选择常见的拼写错误来替换它。
2.把它的一些字符修改成类似的外观。比如把 l(el) 改成 1(一) 。


<div align=center>
  <img width=472 src="https://vikingstudyhard.github.io/images/Paper_DeepTextClassificationCanBeFooled/2FD629E029DE461B239BFBD260134517.jpg" >
</div>

修改之后要分别计算上面两种损失函数的梯度，看看是否朝期待的方向发展。

短语 “*comedy film property*”是原始类（Film）的HSP，错字“*flim*”是从一个拼写错误语料库获得的。

<div align=center>
  <img width=519 src="https://vikingstudyhard.github.io/images/Paper_DeepTextClassificationCanBeFooled/6D0C3496E6E5225F6C8E0F434D4BFAA9.jpg" >
</div>

化工业、公司有关

这样可以使文本被分为company类（99.0%）。

### 移除策略

<div align=center>
  <img width=479 src="https://vikingstudyhard.github.io/images/Paper_DeepTextClassificationCanBeFooled/1D2F721388E46A8273BE5985D53FEAA7.jpg" >
</div>

被分为film类（95.5%）。计算梯度后发现“*seven-part British television series*”是HSP，对分类最有贡献。移除“*British*”后置信度降到60.5%。

## 黑盒方案
在黑盒方案中，由于无法直接得到模型的详细信息，作者借助了模糊测试技术，用一个 **空格序列**逐个遮挡句子中的单词再进行分类，如果 **分类结果偏差**越大，则说明被遮挡的单词对正确的分类越重要，这样就能确定HTP和HSP。

<div align=center>
  <img width=503 src="https://vikingstudyhard.github.io/images/Paper_DeepTextClassificationCanBeFooled/124482504CF2E9B396EA16EDA5B6B967.jpg" >
</div>

# 实验
在实验中，作者对不同的对抗场景进行白盒或黑盒攻击，选择了字符级模型[5]和单词级模型[6]两个代表性的文本分类DNN作为攻击目标，制作文本对抗样本。

在字符级模型的实验中，作者对DB-pedia[4]本体数据集进行训练。在单词级模型的实验中，作者使用了影评数据库MR[1]，产品评论CR[2]，情感信息语料库MPQA[3]三个数据集。

# 评估
(结果如何，如何验证解决效果，介绍本文的实验数据，与其他同类工作的对比方法，优缺点，创新点)

<div align=center>
  <img width=473 src="https://vikingstudyhard.github.io/images/Paper_DeepTextClassificationCanBeFooled/6B60866AC8FFD80BFC5C93C6CFA844CD.jpg" >
</div>

通过多类数据集DB-pedia进行实验，结果表明该方法可以有效地进行源/目标的错误分类攻击。1-13说明了可以fool到任意类中。14-16说明了任意类的文本也能被fool。

对23名学生进行单盲试验，要求他们手动对混有原始样本和对抗样本的20个样本进行分类，如果他们认为样本是人为修改的，则要求他们确定修改的执行位置。对于十个原始样本，受试者将其分类为平均准确度为94.2％，而对于十个受影响的样本，准确度为94.8％。结果表明生成的对抗样本很难被察觉。


在效率上
- 大数据集（DBpedia数据集）
    - 白盒攻击：计算成本梯度并识别14个类的HTP，共116个小时（每类8.29小时）
    - 黑盒攻击：生成测试样本和识别HTP，需要107小时（每个类7.63小时）
- 小数据集（MR，CR和MPQA）
    制作一个对抗样本大约需要15分钟

黑盒白盒获得的HTP高度重叠，其中约80％是相同的。对于给定样本，获得的HSP通常是相同的。此外，在实践中，白盒分析可以识别在黑盒测试中遗漏的一些热词，反之亦然。




# 但是
检测正确的短语并使用它们来创建对抗性样本需要一些启发式方法，文中没有非常清楚地提到这些方法。

也没有代码

但也提供了很好的思路



# Reference

[1] Bo Pang and Lillian Lee. Seeing stars: Exploiting class relationships for sentiment categorization with respect to rating scales. In Proceedings of the 43rd annual meeting on association for computational linguistics, pages 115–124. Association for Computational Linguistics, 2005.
[2] Minqing Hu and Bing Liu. Mining and summarizing customer reviews. In Proceedings of the tenth ACM SIGKDD international conference on Knowledge discovery and data mining, pages 168–177. ACM, 2004.
[3] Janyce Wiebe, Theresa Wilson, and Claire Cardie. Annotating expressions of opinions and emotions in language. Language resources and evaluation, 39(2-3):165–210, 2005.
[4] Auer, S.; Bizer, C.; Kobilarov, G.; Lehmann, J.; Cyganiak, R.; and Ives, Z. 2007. DBpedia: A nucleus for a web of open data. In Proc. of the 6th International Semantic Web Conference.
[5] Xiang Zhang, Junbo Zhao, and Yann LeCun. Character-level convolutional networks for text classification. In Advances in neural information processing systems, pages 649–657, 2015.
[6] Yoon Kim. Convolutional neural networks for sentence classification. In Proceedings of the 2014 Conference on Empirical Methods in Natural Language Processing (EMNLP), pages 1746–1751. Association for Computational Linguistics, 2014.
