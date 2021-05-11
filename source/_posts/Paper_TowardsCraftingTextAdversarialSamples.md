---

title: 【论文略读】Towards Crafting Text Adversarial Samples

date: 2019.03.08

categories: 
 - [AI testing, 文本对抗样本的生成]

tags: 

 - Deep Learning
 - 论文精读
 - 文本对抗样本的生成

description:  一种贪婪算法
mathjax: true

---

https://arxiv.org/abs/1707.02812
> Samanta, S., & Mehta, S. (2017). Towards crafting text adversarial samples. arXiv preprint arXiv:1707.02812.

# 做什么

Samanta等人提出一种文本的对抗样本生成方法，为了确保语义和句法的保留，作者通过删除或替换文本中的重要或突出单词或在文本样本中引入新单词，并且用贪婪算法考虑每个合适的替换词。

此方法适合于具有**子类别**的数据集。

子类别（sub-categories）：

IMDB影评的情感分析

**词义**

*“good、excellent、like”*等表明了积极的评价，而且这些与电影的类型（the genre of the movie）无关。
*“The movie was hilarious[hɪ'leərɪəs]”* 好笑的，滑稽的。
当电影是喜剧片时 hilarious 代表积极的评价，是恐怖片时 hilarious 代表消极的评价（太搞笑了一点恐怖感都没有），这就与电影的类别有关。


<div align=center>
  <img width=687 src="https://vikingstudyhard.github.io/images/Paper_TowardsCraftingTextAdversarialSamples/EE6EA758BD8C2978CE37A0CFB2F5C33C.png" >
</div>


**副词**

“The movie was fair”——一般的普通的电影——可能属于任何一类情绪
“The movie was *extremely* fair”——这电影也太一般了——负面情绪


# 怎么做

## 算法
样本：$s$
情感分类器：$F$
类别：$y$
每个单词的贡献率：$C_F(\omega_i,y)$
$\omega_i$的候选池：$P$

<div align=center>
  <img width=865 src="https://vikingstudyhard.github.io/images/Paper_TowardsCraftingTextAdversarialSamples/C59988A62F76C23451DEC7985CCB180C.png" >
</div>

**贡献率**
一个单词有“高贡献”是指去掉它后文本将被分为当前类的概率大幅减小。
计算量大，一般借鉴FGSM的方法近似计算。（计算了成本梯度 --> 白盒）

**候选池**
- **同义语**
  good = nice = decent
- **拼写错误**——太显眼——约定：拼错成有效词汇
  good --> god, goods
- **特定类型的关键词**
  hilarious 一类电影中被作为积极评价词，但在另一类电影中被作为负面评价词的单词

## 策略
- 移除单词（removal of word）
- 插入单词（addition of word）
- 替换单词（replacement of word）

这实际上是使用到了贪婪算法（greedy method），争取做最少的修改，同时最大程度地保留句子结构。

# 实验
作者在两个数据集上进行了实验：用于情感分析的IMDB电影评论数据集[1]和用于性别分类的twitter数据集[2]。

# 评估
评估指标：
1.比较 **准确性**：分类器正确标记的样本的百分比
2.测量原始样本与对抗样本之间的 **语义相似性**，只有在成功地制作了对抗性样本之后, 文本样本与其对抗性对应方之间的语义相似性才有效。

用IMBD数据集做情感分析，Textfool[3]相比较

<div align=center>
  <img width=857 src="https://vikingstudyhard.github.io/images/Paper_TowardsCraftingTextAdversarialSamples/B8D1B059BC7BD065CB75613F51AB9784.jpg" >
</div>

> 第3行Accuracy using original test set 是baseline的准确率 用来做对比
第4行Accuracy using adversarial test set 是生成对抗样本的准确率
第3列Proposed method using genre specific keywords 代表 在候选池中考虑“特定类型关键词”时的准确率；
第4列Proposed method w/o using genre specific keywords 代表 在候选池中不考虑“特定类型关键词”时的准确率。
第5行percentage of perturbed samples 代表：成功产生对抗样本的概率。由于语义约束，有的测试样本无法成功产生对抗样本。

**准确性：**

74.53: textfool效果不好 因为原文没有详细说明咋弄的 大家根据理解写的算法 效果并不好
90.64与42.76：包含特定类型的关键字肯定会提高样本制作的质量
retrain：用对抗样本重新训练后，这两行精度的差异非常小。

**语义相似性：**

使用/不使用genre specific keywords时产生的对抗样本和原始样本间的平均语义相似度分别为0.9164和0.9732。

原始及其对应的扰动样本之间的语义相似性确实在候选池中的类型特定关键字减少了一点，
但是在这种情况下生成的有效对抗样本的数量也增加：90.64--42.76

# Reference

[1] A. L. Maas, R. E. Daly, P. T. Pham, D. Huang, A. Y. Ng, and C. Potts. Learning word vectors for sentiment analysis. In Association for Computational Linguistics: Human Language Technologies, 2011.
[2] CloudFlower. Twitter gender classification dataset. https://www.kaggle.com/crowdflower/twitter-user-gender-cl 2013.
[3] Liang, B., Li, H., Su, M., Bian, P., Li, X., & Shi, W. (2017). Deep text classification can be fooled. arXiv preprint arXiv:1704.08006.
