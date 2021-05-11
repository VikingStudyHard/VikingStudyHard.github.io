---
title: CNN 文本分类
date: 2019.7.13
categories: NLP
tags: 
    - pyTorch
    - NLP
description: Text CNN
mathjax: true
---

# 数据集
SST-2 (Stanford Sentiment Treebank): 单句的二分类问题，句子的来源于人们对一部电影的评价，判断这个句子的情感。
数据集包括训练集，开发集，测试集。

训练集(training set)用于运行你的学习算法。
开发集(development set)用于**调整参数**。
测试集(test set)用于评估算法的性能，但不会据此决定使用什么学习算法或参数。**不能在测试集上调参！**

路径都保存到`/config/data_path.json`中。
形式为 `label ||| word0 word1 word2 ... wordn`

> 1 ||| a stirring , funny and finally transporting re-imagining of beauty and the beast and 1930s horror films
0 ||| apparently reassembled from the cutting-room floor of any given daytime soap .
0 ||| they presume their audience wo n't sit still for a sociology lesson , however entertainingly presented , so they trot out the conventional science-fiction elements of bug-eyed monsters and futuristic women in skimpy clothes .
1 ||| this is a visually stunning rumination on love , memory , history and the war between art and commerce .

每行数据用`Instance`对象表示成label与words列表。

`dataloader/Dataloader.py` 中的 `load_dataset` 方法将所有数据以`Instance`对象的形式存入insts列表中，并用numpy中的shuffle打乱insts列表。得到训练集`train_data`，开发集`dev_data`，测试集`test_data`。

**基本步骤**
1. 读取加载数据集
2. 预处理（统一中英文符号，空格，数字……）
3. word2id
4. 生成 Batch tensor

# 生成词表和字符表

`/vocab/Vocab.py` 中的 `create_vocabs` 方法通过Counter类用`wd_counter`和`lbl_counter`以字典的键值对形式存储训练集中每个label和word和它们的出现次数。对于word中的每个char也用`char_counter`记录字符和出现次数。

在 `/vocab/Vocab.py` 的 `WordVocab` 类中，`_wd2freq`过滤低频词，以字典的形式存储word和它的出现次数。
`_wd2idx`生成字典类型{训练集中的词:索引}，并将不在训练集中的词用`<unk>`表示，index设为0。
同理得到：
`_idx2wd`，即{索引:训练集中的词}
`_lbl2idx`，即{训练集中的label:索引}
`_idx2lbl`，即{索引:训练集中的label}

`/vocab/Vocab.py` 中的`get_embedding_weight` 方法为了获取预训练词向量的权重。

在词向量文件`/data/word2vec_40w_300.txt`中保存了40w个单词的词向量，词向量大小为300(每行有300个小数)。形式如下：
> in 0.0703125 0.08691406 0.087890625 0.0625 0.06933594 ... -0.0625
for -0.011779785 -0.04736328 0.044677734 0.06347656 -0.018188477  ... 0.024169922

保存到`vec_tabs`字典中，{词:词向量array}，如{'in':[0.0703125 0.08691406 0.087890625 0.0625 0.06933594 ... -0.0625]}
按照之前的方法，得到`_extwd2idx`，即{词向量文件中的词:索引}，将不在词向量文件中的词用`<unk>`表示，index设为0。
以及`_extidx2wd`，即{索引:词向量文件中的词}。

用`oov_ratio`计算得到：训练语料库中不在词向量文件的词，占整个训练语料库的词的比例。


`embedding_weights` 建立一个[词向量文件中词的个数 $\times$ 向量大小]的矩阵，即40w+1 $\times$ 300。
通过词向量文件得到idx对应的词向量，idx为0指的是`<unk>`，它的向量随机赋值。

{% img https://vikingstudyhard.github.io/images/NLP_TextCNN/8F63A529A6E29823F86333772F07939B.png 800 %}

用`save`方法通过pickle保存词表 

字符和label无关，也没有什么向量文件。在 `/vocab/Vocab.py` 的 `CharVocab` 类中，`_ch2idx`生成字典类型{训练集中的字符:索引}，并将不在训练集中的字符用`<unk>`表示，index设为0。
同理得到：`_idx2ch`，即{索引:训练集中的字符}


# 构建textCNN模型
在`model/testcnn.py`的`TextCNN`类中，`embed_dim` 用 `shape` 获取 `embedding_weights` 第二维度长度，即词向量大小。
`from_numpy`把array的`embedding_weights`转化为tensor。`from_pretrained`创建词向量对象embed。


`_win_size`将窗口大小设为3，4，5。也就是3-gram，4-gram，5-gram。`_convs`将根据不同的`kernel_size`建立不同的卷积层、激活层、池化层。
 
卷积层使用一维卷积`Conv1d`函数。
`in_channels`是词向量大小`embed_dim` 
`out_channels`是卷积核的个数，若使用固定值可以设置为`args.hidden_size`，若使用可变值可以设置为`kernel_size`的倍数
`kernel_size`是卷积核的大小，也就是滑动窗口的大小，从数列`_win_size`中获取。
`padding`是数据填充。

激活层使用`ReLU`函数。

池化层使用`AdaptiveMaxPool1d`自适应池化层，只设置输出为1，避免了设置输入。也就是对全局做池化。


dropout层只在训练时出现，共有两层，分别在卷积之前和线性层之前。作用是防止过拟合。
所谓的置为0的概率是指**每个节点置为0的概率**，不是对所有节点置为0的概率。

线性层 y=wx+b 输出类别。

{% img https://vikingstudyhard.github.io/images/NLP_TextCNN/B22C5B2273BDC522295B5988B0679E78.png 1000 %}
（图上显示的只有词表时的情况，加上字符表后情况类似。。）

输入word： wd_idx [batch_size, seq_len] (指的是max_seq_len)
通过`from_pretrained`得到词向量。
词向量 embed：[batch_size, seq_len, word_embed_dim]

同理，输入char： ch_idx [batch_size, seq_len, word_len] (指的是max_seq_len，和max_word_len)
词向量 embed：[batch_size, seq_len, char_embed_dim]

将两种embed沿着第3维度拼接在一起，得到总embed： [batch_size, seq_len, embed_dim] 

如果当前是训练，则需要dropout。
embed 经过transpose转置得到：[batch_size, embed_dim, seq_len]
经过卷积层Conv1d、激活层、池化层得到：[batch_size, hidden_size, 1]
沿着hidden_size进行拼接得到：[batch_size, hidden_size $\times$ len(self._win_size), 1]。squeeze后得到conv_out：[batch_size, hidden_size $\times$ len(self._win_size)]
如果当前是训练，则需要dropout。conv_out： [batch_size, hidden_size $\times$ len(self._win_size)]
经过线性层得到：[batch_size, label_size]

# 分类器
## 训练与验证
输入为train_data和dev_data
### 训练
epoch：整个数据集训练几轮
batch：一次训练多少句话

`/Dataloader.py`中的`get_batch`方法获取批量数据`batch_data`，每条数据是`Instance`对象，表示成label与words列表。`batch_variable`方法找到每组batch_data句子的个数`batch_size`，每组batch_data中的最大长度`max_seq_len`。
文本有长有短，需要和最长的句子对齐。因此建立[batch_size, max_seq_len]的`wd_idx`和`mask`，以及[batch_size]的`lbl_idx`。

以上是词的处理方法，接下来是对齐字符。每个词中字符的个数也不同，需要与最大的对齐。得到`max_char_len`，因此建立[batch_size, max_seq_len, max_char_len]的`ch_idx`。


在`/Dataloader.py`中的`batch_variable`方法中，对于`batch_data`中的数据，通过枚举类型同时列出insts和insts的下标i（i表示第几句话），通过`extwd2idx`判断`inst.words`是list还是单个单词，并通过`_extwd2idx`得到词在词向量文件中对应的index，然后转化为tensor。用`seq_len`表示数据`inst.words`的长度，即数据中单词的个数。`wd_idx`的第i行前`seq_len`个单词，分别对应tensor形式的索引。

同理，通过`char2idx`判断`inst.words`中的下标为j的单词`wd`（j表示一句话中第几个词）是字符list还是单个字符，并通过`_ch2idx`得到字符对应的index，然后转化为tensor。用`len(wd)`表示当前单词的长度，即单词中字符的个数，则`ch_idx`的第i句第j个词前`len(wd)`个字符，分别对应tensor形式的索引。

`lbl_idx`的第i个对应此insts的label在语料库中的index的tensor形式。
`mask`和`wd_idx`相似，将第i句话所有有单词的位置置为1。



在`classifier.py`中的`train`方法通过`batch_variable`数据变量化，得到`wd_idx`，`ch_idx`，`lbl_idx`和`mask`。
通过`zero_grad`将模型梯度置为零，前向传播将数据`wd_idx`，`ch_idx`和`mask`喂给模型，得到预测值pred：[batch_size, label_size]。

`_calc_loss`通过CrossEntropyLoss计算预测值与正确答案之间的误差loss。


> CrossEntropyLoss
针对单目标分类问题, 结合了`nn.LogSoftmax()` 和 `nn.NLLLoss()` 来计算 loss。
在数学中，交叉熵
$$
H(P, Q)=-\sum_{y} P(y) \log Q(y)
$$
P(y) 是onehot，如[0,1,0]表示第二类。$\log Q(y)$表示概率或score。
$$
\operatorname{softmax}(Q)=\frac{e^{Q}}{\sum_{i} e^{Q_{i}}}
$$
softmax 使概率和为1，如[0.2, 0.5, 0.3]。取负的对数似然，所以交叉熵有负号。

`_calc_acc`计算准确率，找到pred中最大值对应的类别索引，与正确答案进行比对，统计预测正确的数量。
通过`train_loss`和`train_acc`统计每个batch的损失的数量和正确数量。
用`backward()`对`loss`进行反向传播。
`optimizer.step()`用优化器Adam更新模型参数。

### 验证
`_validate`函数将每一轮epoch的所有batch训练完成后，用开发集调参数。
验证与训练相似，但不再需要梯度。通过`dev_loss`和`dev_acc`统计每个batch的损失的数量和正确数量。



### 计算loss和acc
`train_loss`和`train_acc`除以训练集大小，得到误差的比率和准确率，并分别存入`train_loss_lst`和`train_acc_lst`。
`dev_loss`和`dev_acc`除以训练集大小，得到误差的比率和准确率，并分别存入`dev_loss_lst`和`dev_acc_lst`。
每一轮epoch都要输出`train_loss`和`train_acc`、`dev_loss`和`dev_acc`。

### 早停
防止过拟合
epoch 每经过patience轮，如果开发集的准确率没有上升，（或者开发集的损失值没有下降也行），则终止训练。这个从`dev_acc_lst`比较。


## 评估
输入为test_data
`evaluate`与验证函数`_validate`相似，在最后输出`test_loss`和`test_acc`。

