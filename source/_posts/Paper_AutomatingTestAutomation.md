---

title: 【论文精读】Automating Test Automation

date: 2019.04.06

categories: 
  - [软件测试, 测试自动化]

tags: 
  - 论文精读

description: 作者提出了关键字驱动的自动测试自动化工具ATA，基于回溯的搜索来解决测试人员指令中固有的含糊之处，可以根据自然语言编写的测试用例，生成可自动编译的形式

---

https://dl.acm.org/citation.cfm?id=2337327

> Thummalapenta S, Sinha S, Singhania N, et al. Automating test automation[C]//Proceedings of the 34th International Conference on Software Engineering. IEEE Press, 2012: 881-891.

{% label primary@输入 %}： 一系列自然语言描述的测试步骤


{% img https://vikingstudyhard.github.io/images/Paper_AutomatingTestAutomation/E0A91FA3DAD976E5BAEF5F0B09B758B7.jpg 424 %}

{% label primary@输出 %}： 一系列过程调用

{% img https://vikingstudyhard.github.io/images/Paper_AutomatingTestAutomation/BA88EA134E1E95A6A3635721383986EB.jpg 427 %} 

一系列自然语言编写的测试步骤，根据连接词分割出初步细分段，在每个段中提取动词作为action，动词作用的名次对象作为target，相关的数据作为data。

{% img https://vikingstudyhard.github.io/images/Paper_AutomatingTestAutomation/000B86984D72943BD24E7ADE1019C17B.jpg 424 %} 


{% label danger@ambiguity不明确 %}

1. 根据连接词分段
`enter login guest and password guest`不能分成`enter login guest`和`password guest`。
2. 一个段中三元组可能有许多种情况
如上图中第三列，一个段由好几种可能的候选元组。人类可以凭直觉选择正确的情况，但机器不能。
3. 段是没意义的（spurious），对应不需要执行的操作
`Drop down list`要做的事情已经在之前的段`select Category as All`里完成了
或者target本身不明确，如一个页面由多个ok按钮

通过非确定性程序对一系列模糊测试步骤进行建模，其中捕获了每个测试步骤的所有可能的替代含义。该非确定性程序的一些确定是所需测试步骤的正确顺序。然后将问题简化为找到非确定性程序的正确确定之一。我们的方法通过回溯探索来实现这一点，同时尝试针对应用程序解释测试步骤。该应用程序充当测试oracle。我们继续探索一种替代方案，直到可以解释整个测试用例或者解释达到无法继续的程度。

# 做什么
通过非确定性（non-deterministic）程序对一系列模糊（ambiguou）测试步骤进行建模，其中，每个测试步骤的所有可能的含义都会被捕获到。通过回溯探索，找到正确的确定化操作（determinisation），并输出对应的action, target, data三元组序列。
# 怎么做

深度优先遍历测试流程图（test-flow graph）寻找从根结点到叶子结点可行的路线。


**manual test case**: 一系列test steps。
`Enter login guest and password guest, and click login`

**segment list**: 根据`and, from, by, then, under, on`等连接词拆分成segment。
```
Seg 1: enter login guest
Seg 2: password guest
Seg 3: click
```
有四种可能：SL 1, SL 2, SL 3, and SL 4.

**tagged segment**: 词性标注
```
TS 1: enter login/verb guest
TS 2: enter login/noun guest
```

**disambiguated tuple**: 消歧元组，与UI元素相对应
```
UI 1: login->textbox
UI 2: login->button
```

{% label success@算法 %}
{% img https://vikingstudyhard.github.io/images/Paper_AutomatingTestAutomation/FF276CC264A057244272282776451AC2.jpg 413 %}

{% img https://vikingstudyhard.github.io/images/Paper_AutomatingTestAutomation/22B70DF824C6C3D525CCBA19600ABE41.jpg 418 %}

T2 无效所以回溯
生成元组`<enter,login,guest>`最后根据UI元素进行匹配

# 评估方法
1. ATA在没有人为干预的情况下执行手动测试的有效性
2. 三种优化技术在减少所用时间和回溯次数方面所取得的效率
3. 分割的有效性：precision 和 recall


# 局限性
多个可行的执行方法：测试流程图中可能存在多个可行路径。目前，ATA使用它找到的第一个可行路径计算自动脚本。然而，ATA可以很容易地适应于通过不终止而产生所有可行路径，而是在找到第一个可行路径之后继续进一步继续，并且在候选可行解释中将正确解释的分辨率留给人类判断。

状态恢复：在回溯期间探索新的替代方案时，ATA将应用程序的状态恢复到每个决策节点的先前状态。目前，ATA仅支持应用程序级状态恢复 - 它无法恢复后端数据库的持久状态。在我们的评估中，由于此限制，ATA无法自动执行三个手动测试（一个在BookStore中，另一个在BugTracker中）。在将来的工作中，我们计划通过使用数据库级检查点和回滚来恢复持久状态来解决此限制。

ATA生成的脚本的正确性：只有在所有验证步骤都通过后，ATA才会生成测试脚本，从而高度确信生成的脚本的正确性。但是，我们建议ATA的用户通过回放脚本来验证最终脚本，以防止ATA选择也通过所有验证步骤的错误可行路径的罕见概率。

