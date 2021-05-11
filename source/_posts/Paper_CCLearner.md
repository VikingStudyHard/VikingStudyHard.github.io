---

title: 【论文精读】CCLearner：A Deep Learning-Based Clone Detection Approach

date: 2019.7.11

categories: 
 - [代码克隆]

tags: 
 - Code Clone
 - 论文精读


description: 第一个使用深度学习的基于token的克隆检测方法：CCLearner
mathjax: true

---
> Li L, Feng H, Zhuang W, et al. Cclearner: A deep learning-based clone detection approach[C]//2017 IEEE International Conference on Software Maintenance and Evolution (ICSME). IEEE, 2017: 249-260.

# 做什么
CCLEARNER 是第一个使用深度学习的基于token的克隆检测方法。
# 怎么做
{% img https://vikingstudyhard.github.io/images/Paper_CCLearner/FD8228EC68B832B18CF8D88EEB40CFBD.jpg 771 %}

## 特征

对于每个给定的方法对 $(method_A，method_B)$ ，首先使用 ANTLR lexer 对代码进行令牌化，以识别每种方法中使用的所有 token，然后记录每个 token 的出现次数，组成<key, value>对。将<key, value>对的序列作为每个方法的 token-frequency list，表示为方法 A 的 token_freq_listA 和方法 B 的 token_freq_listB。最后将token进行分类并计算特征作为 token 使用的相似性得分。

CCLEARNER将 token 分为八类：
{% img https://vikingstudyhard.github.io/images/Paper_CCLearner/7363CFA91ACDA6D8E7D26EF7F20ECB44.jpg 435 %}
### 特征提取
#### C1-C3——ANTLR 词法分析器

由于一型克隆到三型克隆几乎总是具有相同的程序句法结构，可能包含稍微不同的算术或逻辑运算。所以C1-C3的 token-frequency list 可以很容易地从 ANTLR 词法分析器中输出构建。但对于C4-C8 token，ANTLR并不能总是精确地**识别**出token或清楚地**区分**出各种类型的token。（如：-1只能识别出1；不能判断标识符foo表示的是类型还是变量）

#### C4-C8——Eclipse ASTParser

为了精确识别C4-C8，CCLEARNER 还使用 Eclipse ASTParser 为每个方法创建**抽象语法树**（AST），并通过访问某些类型的AST节点实施独立的ASTVisitors，以提取不同类型的 token。

{% img https://vikingstudyhard.github.io/images/Paper_CCLearner/917DE7F8BA61CF7F1BEA151DA877B24E.jpg 451 %}

但是 ASTParser 不会代替 CCLEARNER 中的 ANTLR，因为 ASTParser 无法显示 ANTLR 检测到的所有 token。

为了识别常量（Literals，即C4 token），使用 ASTVisitor 访问所有常量 AST节点（例如，BooleanLiteral 和 CharacterLiteral），并在 token-frequency list 中为每个唯一常量创建条目。对于图3中的 AST，当访问NumberLiteral `1`时，CCLEARNER 进一步检查文字是否是负数的子表达式（即-1）；如果是，则提取负数。通过这种方式区分正数和负数。


对于类型标识符（Type identifiers，即C5 token），使用 ASTVisitor 访问所有与类型相关的AST节点，如 PrimitiveType，TypeLiteral 等。对于图3中的 AST，当访问 PrimitiveType `int`时，CCLEARNER 为类型名称创建一个条目，然后计算名称的出现次数。

为了提取方法标识符（Method identifiers，即 C6 token），使用 ASTVisitor 访问所有与方法相关的AST节点，例如 MethodDeclaration 和 MethodInvocation，可以精确地识别方法实现中使用的所有方法名称。对于图3中的 AST，有一个MethodInvocation元素，因此 CCLEARNER 提取方法名称`foo`。

限定名称（Qualified names，即C7 token）是特殊 token，创建一个 ASTVisitor 来特别处理所有 QualifiedName 节点。给定像`Foo.Goo.Bar`这样的限定名称，我们可以将其解析为类型标识符或变量标识符，我们也可以将名称的一部分`Foo.Goo`解析为类型或变量。使用从方法的AST派生的有限程序句法信息，我们不总能精确地确定限定名称是否代表类型。虽然我们可以在限定名称上调用`resolveBinding()`API，但调用并不总是有助于解析名称绑定。为了避免对类型或变量标识符进行任何不准确的分类，我们因此简单地将限定名称视为单独的标记类别。对于图3中的示例，`a.b`被提取为限定名称，虽然我们不知道它表示类型还是变量。

变量标识符（Variable identifiers，即C8 token）比上面的令牌更难提取，因为没有固定的AST节点类型集可能在特定位置包含这样的标识符。因此，创建了一个 ASTVisitor 来访问所有 SimpleName 节点，并排除属于上述任何类别的标记所涵盖的SimpleName。在示例AST中，有四个SimpleName元素：`v`，`a`，`b`和`foo`。由于`a`和`b`被限定名称`a.b`覆盖，而`foo`已被识别为方法名称，因此CCLEARNER最终以这种方式识别一个变量标识符`v`。

### 创建向量
如下图所示，在提取每个方法分类后的token后，CCLEARNER创建了 token-frequency lists 的向量，表示为 $[token\\_freq\\_cat_{A1} ,...,token\\_freq\\_cat_{A8}]$用于方法A，$[token\\_freq\\_cat_{B1} ,...,token\\_freq\\_cat_{B8}]$用于方法B。

### 计算相似性得分
两种方法被表征为令牌频率列表的向量时，我们认为向量之间的相似性应该反映其方法的相似性。直觉上，两个矢量越相似，这两种方法相互克隆的可能性就越大。相应地，如果两种方法不相似，则它们的向量是不同的。因此，对于每个方法对，CCLEARNER 计算每个标记类别的 token-frequency list 之间的相似性得分，然后构造八个值的相似性向量以表征方法对关系。

形式上，对于类别 $C_i(1\leq i\leq 8)$，如果将两个术语频率列表表示为 $L_{A_i}$ 和 $L_{B_i}$，则相似性得分 $sim\\_score_i$ 计算如下：
$$
sim\_score_{i}=1-\frac{\sum_{x}\left|freq\left(L_{A i}, x\right)-freq\left(L_{B i}, x\right)\right|}{\sum_{x}\left(freq \left(L_{A i}, x\right)+freq\left(L_{B i}, x\right)\right)},
$$

$$
\text{where }x \in tokens\left(L_{A i}\right) \cup tokens\left(L_{B i}\right).
$$
直觉上，我们枚举了两个列表中任何一个中包含的所有token。对于每个枚举的token $x$，我们计算其**绝对频率差值** $(\left|freq\left(L_{A i}, x\right)-freq\left(L_{B i}, x\right)\right|)$和**频率和**$\left(freq \left(L_{A i}, x\right)+freq\left(L_{B i},x\right)\right)$。接下来，我们分别对所有**绝对频率差值**和所有**频率和**进行求和以计算比率，从1中减去该比率以获得相似性得分。根据该公式，相似度得分范围为$[0,1]$。

我们的相似度计算公式很有意义。当两种方法相同且具有完全相同的token-frequency list 时，**绝对频率差和**与**频率和**的比率始终为0，因此每个类别的相似度得分为1。当两种方法完全不同且不共享token时，相应的比率为1，而相似性得分为0。

假设我们有两个token-frequency list：$L_A = {\langle a,3 \rangle,\langle b,4 \rangle,\langle c,5 \rangle}$ 和 $L_B = {\langle b,3 \rangle,\langle c,6 \rangle,\langle d,2 \rangle}$。相似度得分计算为$1-(|3-0| + |4-3 |+ |5-6| +| 0-2 |)/((3 + 0)+(4 + 3)+(5+ 6)+(0 + 2))= 0.70$。一般来说，列表之间共享的token越多，每个token的频率差越小，我们可以得出的相似度得分越高。请注意，如果两个方法没有特定类别的token（例如关键字），我们默认将相应的相似性分数设置为0.5。

## 训练
为了训练克隆检测二分类器，我们需要克隆关系的正面和负面示例的特征数据。正例是从克隆方法对中提取的特征向量，而负例是从非克隆对中得到的向量。在我们的训练数据中，每个数据点具有以下格式：$\langle similarity\\_vector,label\rangle$，其中similarity_vector是有八个相似性得分的向量，$label$取值1或0，1表示“CLONE”，0表示“NON_CLONE”。为了避免由小克隆方法引起的任何噪声，有意过滤掉包含少于6个LOC（lines of code）的方法来构建训练数据。

使用DNN的开源库 DeepLearning4j 来训练分类器。由于定义了八个特征，我们为输入层配置了八个节点，每个节点独立地接受一个特征值。由于分类任务有两个标签：“CLONE”和“NON_CLONE”，输出层包含两个节点以显示DNN的分类结果。


## 测试

给定代码库，CCLEARNER首先使用 Eclipse ASTParser 从源代码文件中提取方法，然后枚举所有可能的方法对。 CCLEARNER 将每个枚举的方法对提供给训练的分类器，以确定方法是否为克隆。如果两种方法被判断为克隆，则 CCLEARNER 会相应地报告它们。

输出层有两个节点分别预测克隆和非克隆的可能性：$l_c$和 $l_{nc}$，其中$l_{c} + l_{nc} = 1$.为了将连续似然值映射到“CLONE”或“NON_CLONE”离散分类，我们要求$l_{nc} \leq0.98$以精确检测克隆而不会产生许多误报。

# 评估方法
数据集：BigCloneBench
真实克隆的类别：T1, T2, VST3 (Very Strong Type 3), ST3 (Strong Type 3), MT3 (Moderately Type 3), or WT3/4 (Weak Type 3 or Type 4)

为了评估 CCLEARNER 克隆检测的有效性，将我们的方法与三种流行的克隆检测工具进行了比较：SourcererCC，NiCad 和 Deckard。三个工具都是在 CCLEARNER 的测试数据上使用默认参数配置执行的。与CCLEARNER一样，SourcererCC 根据从源代码中提取的 token 检测克隆。 NiCad 规范化代码并逐行比较代码。 Deckard 比较代码的 AST 以报告具有相似树结构的克隆。我们选择这些工具是因为它们被广泛使用，并且可以很好地代表主流类型的克隆检测方法：基于 token 和基于树。

从四个方面对所有工具进行了比较：Recall，Precision，C score 和 Time Cost。

与现有的基于token的方法相比，CCLEARNER 检测到更多样化的克隆，具有高精度和召回率。与基于树的方法相比，CCLEARNER 有效地检测到具有高精度的克隆。
# 局限性

与Deckard相比，CCLEARNER 有效检测到更多的T1-ST3克隆，但未能报告许多MT3和WT3/4克隆。
首先，CCLEARNER 依赖于在不同方法中使用的完全相同的术语来计算相似性向量。当两个克隆方法共享很少的标识符并包含显着不同的程序结构时，CCLEARNER 无法检测克隆关系。其次，根据 BigCloneBench 论文，在每个WT3/T4克隆对中，这两种方法共享不到50％的共同语句。在没有对语法不同但语义等效的代码段进行推理的情况下，对于任何基于token的方法来检测这样的克隆是非常具有挑战性的。将来，我们计划为这些专家克隆设计一些补充技术。