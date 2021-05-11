---

title: 【ADAS】Hardware in the Loop

date: 2018.5.7

categories: 自动驾驶仿真测试

tags: 

 - CATARC

description:  HIL，hardware-in-the-loop，硬件在环，可用于自动驾驶汽车的仿真测试。

---
# 介绍
HIL，hardware-in-the-loop，硬件在环。以实时处理器运行仿真模型来模拟受控对象的运行状态。

## 关于 “环”
对于一个控制系统，一定要有的基础元素是控制器和被控对象。

举个例子，一个简单的灯泡，控制灯泡亮灭的开关就是控制器，灯泡本身就是控制对象。当然这个控制系统是单向的，这样的控制系统就叫“开环的”。

现在的先进控制系统多是“闭环的”：被控对象通过传感器把自己的状态传给控制器，控制器根据这些状态信息，通过执行器控制被控对象改变状态，循环往复。

HIL就是解决闭环系统的控制器测试问题的。

HIL的作用是用数学模型来模拟被控对象，跟控制器链接成闭环，“欺骗”控制器，让控制器以为它控制的是一个真实的被控对象，从而达到测试控制器的目的。

## 关于硬件

如果将实际控制器的仿真称为虚拟控制器，实际对象的仿真称为虚拟对象，可得到控制系统仿真的3种形式：
1. 虚拟控制器+虚拟对象=动态仿真系统，是纯粹的系统仿真；
2. 虚拟控制器+实际对象=快速控制原型(RCP)仿真系统，是系统的一种半实物仿真；
3. 实际控制器+虚拟对象=硬件在回路(HIL)仿真系统，是系统的另一种半实物仿真。

假想“你的头和你的手”构成一个控制系统，大脑就是控制器，手就是被控对象，眼睛就是传感器。大脑控制手向左移动一米，眼睛要时刻盯着手，看到底有没有移动一米。

这时为了测试大脑这个控制器的功能是否正常，HIL就可以发挥作用了，HIL就是一个高性能的电脑，它上面运行了复杂的算法（数学模型），现在可以把你的手和眼睛拆掉了只把大脑连接到HIL上，HIL这台电脑会模仿眼睛输出一堆视频信号传输给大脑，让大脑以为它自己还能看得见；HIL又跟大脑控制手动作的接口连接，让大脑以为自己还有手，这样大脑控制手左移1米，这个控制命令就传输到HIL里，HIL就通过内部算法解析，然后输出一个手在左移的动画给大脑，这样大脑就被欺骗了，以为自己还在控制一个真实的手。

这主要可以完成对大脑功能的测试，比如大脑想控制手一万次向左移一米，看其中有多少次能移动到位，用真手测试就把手累抽筋了，用HIL模拟手，只要有电，一直测下去也没问题（自动测试、重复测试）；比如要测大脑控制手去拍钉子，用真手就变一次性的了，用HIL假手测就没问题（极限工况测试），等等。


# 优势

1. 提高安全性（如起重机系统的测试）
2. 提升质量（基于设计过程的模型中，HIL可用于早期设计阶段对控制系统，可以通过测试暴露问题和错误，不然这些会在最后阶段即控制系统和机器整合后才会被检测到；测试自动化）
3. 节省时间（时间越长，错误的代价越大）
4. 节约金钱（测试整个机器很贵）
5. 人为因素（大型机器操作人员的训练仿真，帮助训练人员更快理解机器原理、训练人员操作失误时不会出故障、训练人员在任何天气等条件下都能使用、训练人员能学习到不同场景下的应对措施）

# 技术
## Fieldbus Systems 现场总线系统
![IMAGE](/images/ADAS_HIL/B02CB06EE001E4F66AF38BA59D0C6573.jpg)
缺点：
1. 许多IO部件需要连接所有电线
2. 电线传输的频率特别高（尤其是 MHz）

## Control System 控制系统
控制系统部署在PLC（Programmable Logic Controller）上，用来控制机器和系统。


## HIL Simulator
### 基于PC
Controllab提供的HIL模拟器通常是Windows系统的PC机。
![IMAGE](/images/ADAS_HIL/9A076A142D5C20CEAA0BE138719B8554.jpg)
模拟器环境提供了plant模型到IO的合适连接。

### 高速模拟器
Windows不是实时的操作系统，在以下情况下不是很好：
* 当控制系统高速运行(>100HZ)
* 只有模拟器速度很高时，plant模型才运行得好
* 控制系统和plant模型之间存在时间delay，会使系统不稳定（jitter，抖动）

Controllab提供的基于实时Linux的PC的HIL模拟器，允许fieldbus高速通信，并且有低抖动。

### Bachmann M1

## 用于训练的模拟器
配有3D可视化，并能够很好地代替plant。




# 参考
[Introduction to Hardware-in-the-Loop Simulation  C. Kleijn](http://www.hil-simulation.com/images/stories/Documents/Introduction%20to%20Hardware-in-the-Loop%20Simulation.pdf)



[http://ww2.mathworks.cn/help/physmod/simscape/ug/what-is-hardware-in-the-loop-simulation.html](http://ww2.mathworks.cn/help/physmod/simscape/ug/what-is-hardware-in-the-loop-simulation.html)

[https://en.wikipedia.org/wiki/Hardware-in-the-loop_simulation](https://en.wikipedia.org/wiki/Hardware-in-the-loop_simulation)

[https://www.ni.com/zh-cn/innovations/automotive/hardware-in-the-loop.html](https://www.ni.com/zh-cn/innovations/automotive/hardware-in-the-loop.html)