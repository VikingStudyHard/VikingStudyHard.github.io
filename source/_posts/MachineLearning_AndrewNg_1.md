---
title: 【Andrew Ng】Machine Learning | 1 Introdution

date: 2019.01.17

categories: Machine learning

tags: 
 - Andrew Ng

description: 第一章 Introdution 
---
[吴恩达机器学习_网易云课堂](https://study.163.com/course/introduction.htm?courseId=1004570029&_trace_c_p_k2_=704facdf4abd4d84ba87211ab3d65a2d)

第一章 Introdution 绪论：初识机器学习
# 1 Welcome
Machine Learning
  - Grew out of work in AI
  - New capability for computers
Examples:
  - Database mining 数据挖掘
    Large datasets from growth of automation/web.
    E.g., Web click data, medical records, biology, engineering
  - Applications can't program by hand. 无法手动编程，让计算机自己学习
    E.g., Autonomous helicopter, handwriting recognition, most of Natural Language Processing (NLP), Computer Vision.
  - Self-customizing programs 个性化推荐
    E.g., Amazon, Netflix product recommendations
  - Understanding human learning (brain, real AI).理解人类的学习过程和大脑

# 2 What is Machine learning?
## Machine Learning definition
- Arthur Samuel (1959). 让计算机和自己下几万次棋，观察怎么布局能赢怎么会输，来提高能力。
  Machine Learning: Field of study that gives computers the ability to learn without being explicitly programmed. (an older, informal definition)
- Tom Mitchell (1998) 
  Well-posed Learning Problem: A computer program is said to learn from experience E with respect to some task T and some performance measure P, if its performance on T, as measured by P, improves with experience E. (a more modern definition)
    - E 经验：程序和自己下几万次棋
    - T 任务：下棋
    - P 性能度量：与新对手下棋时赢的概率

> e.g. Suppose your email program watches which emails you do or do not mark as spam(垃圾邮件), and based on that learns how to better filter spam. What is the task T in this setting?
    - T:Classifying emails as spam or not spam.
    - E:Watching you label emails as spam or not spam.
    - P:The number (or fraction) of emails correctly classified as spam/not spam.

## Machine learning algorithms
- Supervised learning 监督学习：我们教机器学习
- Unsupervised learning 无监督学习：让机器自己学习

Others: Reinforcement learning 强化学习, recommender systems 推荐系统.
Also talk about: Practical advice for applying learning algorithms.

# 3 Supervised Learning
## Regression
Example:
![IMAGE](/images/MachineLearning_AndrewNg_1/57B79169D505B8A4E710FCCEB2AD727F.jpg)
fit a straight line to the data, or fit the quadratic function(二次函数) to the data
![IMAGE](/images/MachineLearning_AndrewNg_1/0FCF2D68D024909A779EFDAB56330E49.jpg)


In supervised learning, we are given a data set and already know what our correct output should look like, having the idea that there is a relationship between the input and the output.

监督学习：给出“正确答案”。即，给出房屋大小的数据集，以及每个样本对应的实际价格。算法目的是给出更多正确答案，比如给新房子估价。

这种称作**回归问题 Regression：预测连续的输出值。**

## Classification
Example:
乳腺癌肿瘤大小--恶性or良性

![IMAGE](/images/MachineLearning_AndrewNg_1/CBB42C25D71FA9C7D343D669761F339A.jpg)
对应到数轴上，用O表示benign，用X表示malignant。
这种称作**分类 Classification：预测离散的输出值（如 0 or 1）**

对于多重features：肿瘤大小和年龄等
<div align=center>
  <img width=300 src="https://vikingstudyhard.github.io/images/MachineLearning_AndrewNg_1/57EA0645F0A341B8811334B7C20F2F39.jpg" >
</div>

处理无穷多的属性，在电脑中存储，避免内存溢出：支持向量机 Support Vector Machine

## 监督学习
Supervised learning problems are categorized into "regression" and "classification" problems. 

In a regression problem, we are trying to predict results within a continuous output, meaning that we are trying to map input variables to some continuous function. 

In a classification problem, we are instead trying to predict results in a discrete output. In other words, we are trying to map input variables into discrete categories.

> e.g. You're running a company, and you want to develop learning algorithms to address each of two problems.
Problem 1: You have a large inventory of identical items. You want to predict how many of these items will sell over the next 3 months.(对同一货物有大量库存，预测接下来三个月能卖多少)
Problem 2: You'd like software to examine individual customer accounts, and for each account decide if it has been hacked/compromised.(用软件来检查每一个客户的账户是否被入侵)
Should you treat these as classification or as regression problems?
Treat problem 1 as a regression problem, problem 2 as a classification problem.

# 3 Unsupervised Learning
有监督学习：
<div align=center>
  <img width=300 src="https://vikingstudyhard.github.io/images/MachineLearning_AndrewNg_1/B25E10F961C5121737CD6FF79A32DFCE.jpg" >
</div>
无监督学习：没有label
<div align=center>
  <img width=300 src="https://vikingstudyhard.github.io/images/MachineLearning_AndrewNg_1/36503A8D917F4A5B4159BB647FD2BAB6.jpg" >
</div>

只能从中找到两个不同的簇(cluster)
<div align=center>
  <img width=300 src="https://vikingstudyhard.github.io/images/MachineLearning_AndrewNg_1/9E82A8CAA51B6E338C6935C48780E54F.jpg" >
</div>


## 聚类算法 clustering algorithm 
把一堆数据集按照某个标准或者特征，对相似数据划分为一个类别
如：
- 新闻网站搜集新闻，自动分到不同的新闻专题
- DNA 检测特定基因，分类不同人群
- 大型计算机集群
- 社交圈分析
- 市场分割，将客户细分到不同的市场
- 天文学数据分析

## 非聚类算法
从数据集一堆数据中发现相同数据结构
Cocktail party problem
两个话筒同时接收两个不同位置的音源，然后算法进行聚类，提取出每个单独的音源。
`[W,s,v] = svd((repmat(sum(x.* x,1),size(x,1),1).*x)*x');`
svd函数：Singular Value Decmposition 奇异值分解


>  e.g. Of the following examples, which would you address using an unsupervised learning algorithm? 
- Given email labeled as spam/not spam, learn a spam filter.（垃圾过滤——有监督学习）
- Given a set of news articles found on the web, group them into set of articles about the same story.（新闻聚类——无监督学习）
- Given a database of customer data, automatically discover market segments and group customers into different market segments.（自动发现细分市场——无监督学习）
- Given a dataset of patients diagnosed as either having diabetes or not, learn to classify new patients as having diabetes or not.（分类是否糖尿病——有监督学习）


## 无监督学习
Unsupervised learning allows us to approach problems with little or no idea what our results
should look like. We can derive structure from data where we don't necessarily know the
effect of the variables.（不太知道分哪几类）

We can derive this structure by clustering the data based on relationships among the variables
in the data.

With unsupervised learning there is no feedback based on the prediction results.