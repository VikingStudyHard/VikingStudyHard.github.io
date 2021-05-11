---

title: 【软件测试上机】4 Java PathFinder

date: 2018.5.3

categories: [软件测试, 本科课程]

tags: 

 - SoftwareTest
 - 软件测试上机

description: Find and report the division by zero in the following program with Java Pathfinder (JPF)

---

# 要求
• Find and report the division by zero in the following program with Java Pathfinder (JPF)
• Reference
 参考：[http://babelfish.arc.nasa.gov/trac/jpf/wiki](http://babelfish.arc.nasa.gov/trac/jpf/wiki)

```java
public class Racer implements Runnable { 
  int d = 42;
  public void run () {
    doSomething(1001);
    d = 0; // (1) 
  }
  public static void main (String[] args){ 
    Racer racer = new Racer(); 
    Thread t = new Thread(racer); 
    t.start();
    
    doSomething(1000);
    int c = 420 / racer.d; // (2) 
    System.out.println(c);
  }
  static void doSomething (int n) {
    try { 
      Thread.sleep(n);
    } catch(InterruptedException ix) {} 
  }
}
```
完成形式:
• 分组Presentation，每组不超过3个人
• 文档，不低于2000字的文档报告 

展示日期:
  2018.5.1-5.5

其它要求:
• 阅读相关领域的文章
• 进行实证研究
• 有可能的话申报大创项目和软件工程相关的重要比赛

# JPF
参考：
[what is jpf](http://babelfish.arc.nasa.gov/trac/jpf/wiki/intro/what_is_jpf)
[jpf](http://javapathfinder.sourceforge.net/)

JPF是一个Java虚拟机，它和普通的虚拟机一样会执行程序，在理论上还会通过各种可能的路径，检查在所有潜在的执行路径上的死锁或未处理的异常。如果发现了一个error，JPF会报告导致error的整个执行情况。与普通的调试器不同，JPF会跟踪每一步。Java PathFinder 特别适合在多线程的程序中发现很难测试的并发缺陷。

## JPF功能

JPF可以从程序中辨认执行方式可能不同的点，然后系统地探索所有点。也就是说，JPF（理论上）执行程序中的所有路径，不仅仅是像普通VM那样。典型的选择是不同的调度序列或随机值，但JPF也允许您引入您自己的选择类型，如用户输入或状态机事件。

路径的数量可以且通常会无限增长，这就是软件模型检查调用状态爆炸问题的原因。JPF采用的第一道防线是状态匹配：每次JPF到达一个选择点时，它检查它是否已经看到了一个类似的程序状态，在这种情况下，它可以放心地放弃这条路径，回溯到之前的仍有其他未被探索的选择的选择点，并从那里开始继续执行。JPF可以恢复程序状态，就像告诉调试器“返回100条指令”一样。

## 准备工作
参考： [http://babelfish.arc.nasa.gov/trac/jpf/wiki/install/requirements](http://babelfish.arc.nasa.gov/trac/jpf/wiki/install/requirements) 
### 配置环境
VMware Fusion专业版10.0.1
CentOS Linux 7

### Mercurial
参考： [https://www.mercurial-scm.org/wiki/ChineseTutorial](https://www.mercurial-scm.org/wiki/ChineseTutorial)
如果要下载JPF的相关组件，必须要使用一个名叫Mercurial的分布式版本控制系统。
` sudo yum install mercurial `

### Ant
Ant是一个将软件编译、测试、部署等步骤联系在一起加以自动化的一个工具，大多用于Java环境中的软件开发。ant的创始人James Duncan Davidson说，ant代表："Another Neat Tool"。同样的，ant在英文中是“蚂蚁“的意思，这又代表着它有建筑本领高超和身躯虽小，但功能却极其强大。通过Ant我们就可以用命令行进行操作，否则，就要使用NetBeans 或者 Eclipse 。

执行 `sudo yum install ant` ，这时$ANT_HOME的路径就已经配置好了。

### jdk
查看JDK版本，JDK已安装。
```
[Viking@localhost ~]$ java -version
openjdk version "1.8.0_161"
OpenJDK Runtime Environment (build 1.8.0_161-b14)
OpenJDK 64-Bit Server VM (build 25.161-b14, mixed mode)
```

### jpf-core
使用Mercuial下载jpf-core的仓库，执行下列命令
```
$ hg clone http://babelfish.arc.nasa.gov/hg/jpf/jpf-core
$ cd jpf-core
$ ant test
```

出现报错
```
BUILD FAILED
/home/Viking/jpf-core/build.xml:422: Problem: failed to create task or type junit
Cause: the class org.apache.tools.ant.taskdefs.optional.junit.JUnitTask was not found.
        This looks like one of Ant's optional components.
Action: Check that the appropriate optional JAR exists in
        -/usr/share/ant/lib
        -/home/Viking/.ant/lib
        -a directory added on the command line with the -lib argument

Do not panic, this is a common problem.
The commonest cause is a missing JAR.

This is not a bug; it is a configuration problem


Total time: 16 seconds
```

解决方法：
运行 ` sudo yum install ant-junit `

继续出现报错：
```
BUILD FAILED
/home/Viking/jpf-core/build.xml:422: /home/Viking/jpf-core/${env.JUNIT_HOME} does not exist.
```
问题原因：缺少junit.jar文件

### JUnit
把junit-4.12.jar放入/home/Viking/lib里，设置环境变量。
```
$ sudo vi /etc/provile
export JUNIT_HOME=/home/Viking/lib
export CLASSPATH=$CLASSPATH:$JUNIT_HOME/junit-4.12.jar:.
$ source /etc/provile
```
再到jpf-core下执行 ` ant test `
得到下图
![jpf1.jpg](/images/ST_Lab4JPF/E66C856E495A9CB98F34DCAF04E68CCE.jpg)

## 编辑并运行相关代码
参考： [http://babelfish.arc.nasa.gov/trac/jpf/wiki/user/run](http://babelfish.arc.nasa.gov/trac/jpf/wiki/user/run)

### 配置jpf
在jpf-core下新建Racer.jpf
```
target=Racer
listener=gov.nasa.jpf.listener.PreciseRaceDetector
report.console.property_violation=error,trace
```


### 在jpf-core下新建Racer.java
```java
public class Racer implements Runnable { 
  int d = 42;
  public void run () {
    doSomething(1001); // (1) 
    d = 0; // (2) 
  }
  public static void main (String[] args){ 
    Racer racer = new Racer(); 
    Thread t = new Thread(racer); 
    t.start();
    
    doSomething(1000); // (3) 
    int c = 420 / racer.d; // (4) 
    System.out.println(c);
  }
  static void doSomething (int n) {
    try { 
      Thread.sleep(n);
    } catch(InterruptedException ix) {} 
  }
}
```

### 运行jpf

执行 `~/jpf-core/bin/jpf Racer.jpf` 或 `java  -jar /build/RunJPF.jar Racer.jpf`

##jpf输出
jpf有三种输出：
- 应用程序的输出：应用在做什么？
- JPF logging：JPF在做什么？
- JPF 报告：JPF的运行结果是什么？

其中JPF报告如下图所示：
![jpf2.jpg](/images/ST_Lab4JPF/ECDF2A301B772D299B8C1736C135B497.jpg)
具体为：
```
JavaPathfinder core system v8.0 (rev 32) - (C) 2005-2014 United States Government. All rights reserved.


====================================================== system under test
Racer.main()

====================================================== search started: 18-4-24 下午10:56
10
10

====================================================== error 1
gov.nasa.jpf.listener.PreciseRaceDetector
race for field Racer@15b.d
  main at Racer.main(Racer.java:35)
		"int c = 420 / racer.d;               // (4)"  READ:  getfield Racer.d
  Thread-1 at Racer.run(Racer.java:26)
		"d = 0;                               // (2)"  WRITE: putfield Racer.d


====================================================== trace #1
------------------------------------------------------ transition #0 thread: 0
gov.nasa.jpf.vm.choice.ThreadChoiceFromSet {id:"ROOT" ,1/1,isCascaded:false}
      [3157 insn w/o sources]
  Racer.java:30                  : Racer racer = new Racer();
  Racer.java:19                  : public class Racer implements Runnable {
      [1 insn w/o sources]
  Racer.java:21                  : int d = 42;
  Racer.java:30                  : Racer racer = new Racer();
  Racer.java:31                  : Thread t = new Thread(racer);
      [145 insn w/o sources]
  Racer.java:31                  : Thread t = new Thread(racer);
  Racer.java:32                  : t.start();
      [1 insn w/o sources]
------------------------------------------------------ transition #1 thread: 0
gov.nasa.jpf.vm.choice.ThreadChoiceFromSet {id:"START" ,1/2,isCascaded:false}
      [2 insn w/o sources]
  Racer.java:34                  : doSomething(1000);                   // (3)
  Racer.java:41                  : try { Thread.sleep(n); } catch (InterruptedException ix) {}
      [4 insn w/o sources]
------------------------------------------------------ transition #2 thread: 1
gov.nasa.jpf.vm.choice.ThreadChoiceFromSet {id:"SLEEP" ,2/2,isCascaded:false}
      [1 insn w/o sources]
  Racer.java:1                   : /*
  Racer.java:25                  : doSomething(1001);                   // (1)
  Racer.java:41                  : try { Thread.sleep(n); } catch (InterruptedException ix) {}
      [4 insn w/o sources]
------------------------------------------------------ transition #3 thread: 1
gov.nasa.jpf.vm.choice.ThreadChoiceFromSet {id:"SLEEP" ,2/2,isCascaded:false}
      [3 insn w/o sources]
  Racer.java:41                  : try { Thread.sleep(n); } catch (InterruptedException ix) {}
  Racer.java:42                  : }
  Racer.java:26                  : d = 0;                               // (2)
------------------------------------------------------ transition #4 thread: 0
gov.nasa.jpf.vm.choice.ThreadChoiceFromSet {id:"SHARED_OBJECT" ,1/2,isCascaded:false}
      [3 insn w/o sources]
  Racer.java:41                  : try { Thread.sleep(n); } catch (InterruptedException ix) {}
  Racer.java:42                  : }
  Racer.java:35                  : int c = 420 / racer.d;               // (4)
------------------------------------------------------ transition #5 thread: 0
gov.nasa.jpf.vm.choice.ThreadChoiceFromSet {id:"SHARED_OBJECT" ,1/2,isCascaded:false}
  Racer.java:35                  : int c = 420 / racer.d;               // (4)

====================================================== results
error #1: gov.nasa.jpf.listener.PreciseRaceDetector "race for field Racer@15b.d   main at Racer.main(Ra..."

====================================================== statistics
elapsed time:       00:00:00
states:             new=9,visited=1,backtracked=4,end=2
search:             maxDepth=6,constraints=0
choice generators:  thread=8 (signal=0,lock=1,sharedRef=2,threadApi=3,reschedule=2), data=0
heap:               new=362,released=33,maxLive=357,gcCycles=7
instructions:       3424
max memory:         15MB
loaded code:        classes=62,methods=1477

====================================================== search finished: 18-4-24 下午10:56

```


error 抢占资源的类型和详细信息
trace 程序导致这种抢占资源的踪迹
statistics 性能统计信息


## 相关知识
### 多线程
状态图
![jpfthread.jpg](/images/ST_Lab4JPF/90F26580297A213B4F82BF4DFE37D680.jpg)

说明：
线程共包括以下5种状态。
1. 新建状态(New): 线程对象被创建后，就进入了新建状态。例如，Thread thread = new Thread()。
2. 就绪状态(Runnable): 也被称为“可执行状态”。线程对象被创建后，其它线程调用了该对象的start()方法，从而来启动该线程。例如，thread.start()。处于就绪状态的线程，随时可能被CPU调度执行。
3. 运行状态(Running) : 线程获取CPU权限进行执行。需要注意的是，线程只能从就绪状态进入到运行状态。
4. 阻塞状态(Blocked)  : 阻塞状态是线程因为某种原因放弃CPU使用权，暂时停止运行。直到线程进入就绪状态，才有机会转到运行状态。阻塞的情况分三种：
    (01) 等待阻塞 -- 通过调用线程的wait()方法，让线程等待某工作的完成。
    (02) 同步阻塞 -- 线程在获取synchronized同步锁失败(因为锁被其它线程所占用)，它会进入同步阻塞状态。
    (03) 其他阻塞 -- 通过调用线程的sleep()或join()或发出了I/O请求时，线程会进入到阻塞状态。当sleep()状态超时、join()等待线程终止或者超时、或者I/O处理完毕时，线程重新转入就绪状态。
5. 死亡状态(Dead)    : 线程执行完了或者因异常退出了run()方法，该线程结束生命周期。

### Thread和Runnable
参考：[http://www.cnblogs.com/skywang12345/p/3479063.html#part01](http://www.cnblogs.com/skywang12345/p/3479063.html#part01)
常用的实现多线程的2种方式：Thread 和 Runnable。



#### Thread 和 Runnable 的不同点

Thread 是类，而Runnable是接口；Thread本身是实现了Runnable接口的类。我们知道“一个类只能有一个父类，但是却能实现多个接口”，因此Runnable具有更好的扩展性。
此外，Runnable还可以用于“资源的共享”。即，多个线程都是基于某一个Runnable对象建立的，它们会共享Runnable对象上的资源。
通常，建议通过“Runnable”实现多线程。

#### Thread的多线程示例
```java
// ThreadTest.java 源码
class MyThread extends Thread{  
    private int ticket=10;  
    public void run(){
        for(int i=0;i<20;i++){ 
            if(this.ticket>0){
                System.out.println(this.getName()+" 卖票：ticket"+this.ticket--);
            }
        }
    } 
};

public class ThreadTest {  
    public static void main(String[] args) {  
        // 启动3个线程t1,t2,t3；每个线程各卖10张票！
        MyThread t1=new MyThread();
        MyThread t2=new MyThread();
        MyThread t3=new MyThread();
        t1.start();
        t2.start();
        t3.start();
    }  
}
```
这个程序的输出结果是
```java
Thread-0 卖票：ticket10
Thread-1 卖票：ticket10
Thread-2 卖票：ticket10
Thread-1 卖票：ticket9
Thread-0 卖票：ticket9
Thread-1 卖票：ticket8
Thread-2 卖票：ticket9
Thread-1 卖票：ticket7
Thread-0 卖票：ticket8
Thread-1 卖票：ticket6
Thread-2 卖票：ticket8
Thread-1 卖票：ticket5
Thread-0 卖票：ticket7
Thread-1 卖票：ticket4
Thread-2 卖票：ticket7
Thread-1 卖票：ticket3
Thread-0 卖票：ticket6
Thread-1 卖票：ticket2
Thread-2 卖票：ticket6
Thread-2 卖票：ticket5
Thread-2 卖票：ticket4
Thread-1 卖票：ticket1
Thread-0 卖票：ticket5
Thread-2 卖票：ticket3
Thread-0 卖票：ticket4
Thread-2 卖票：ticket2
Thread-0 卖票：ticket3
Thread-2 卖票：ticket1
Thread-0 卖票：ticket2
Thread-0 卖票：ticket1
```
MyThread继承于Thread，它是自定义个线程。每个MyThread都会卖出10张票。主线程main创建并启动3个MyThread子线程。每个子线程都各自卖出了10张票。



#### Runnable的多线程示例

```java
// RunnableTest.java 源码
class MyThread implements Runnable{  
    private int ticket=10;  
    public void run(){
        for(int i=0;i<20;i++){ 
            if(this.ticket>0){
                System.out.println(Thread.currentThread().getName()+" 卖票：ticket"+this.ticket--);
            }
        }
    } 
}; 

public class RunnableTest {  
    public static void main(String[] args) {  
        MyThread mt=new MyThread();

        // 启动3个线程t1,t2,t3(它们共用一个Runnable对象)，这3个线程一共卖10张票！
        Thread t1=new Thread(mt);
        Thread t2=new Thread(mt);
        Thread t3=new Thread(mt);
        t1.start();
        t2.start();
        t3.start();
    }  
}
```
这个程序的输出结果是：
```java
Thread-0 卖票：ticket10
Thread-2 卖票：ticket8
Thread-1 卖票：ticket9
Thread-2 卖票：ticket6
Thread-0 卖票：ticket7
Thread-2 卖票：ticket4
Thread-1 卖票：ticket5
Thread-2 卖票：ticket2
Thread-0 卖票：ticket3
Thread-1 卖票：ticket1
```
和上面“MyThread继承于Thread”不同；这里的MyThread实现了Thread接口。主线程main创建并启动3个子线程，而且这3个子线程都是基于“mt这个Runnable对象”而创建的。运行结果是这3个子线程一共卖出了10张票。这说明它们是共享了MyThread接口的。但是如果没有设置多线程对ticket这个竞争资源的互斥访问，这个例子有可能不止卖出10个票，有可能线程同时访问一个资源。


#### 本题程序

本项目中通过Runnable实现一个接口，从而实现多线程。

### Thread的 start 和 run

start()会启动一个新线程，并在新线程中运行run()方法。而run()则会直接在当前线程中运行run()方法，并不会启动一个新线程来运行run()。

start()源码(基于JDK1.7.0_40)
```java
public synchronized void start(){
	// 如果线程不是"就绪状态"，则抛出异常！
	if(threadStatus != 0)
		throw new IllegalThreadStateException();
	// 将线程添加到ThreadGroup中
	group.add(this);
	boolean started = false;
	try{
		// 通过start0()启动线程,新线程会调用run()方法
		start0();
		// 设置started标记=true
		started = true;
	}finally{
		try{
			if(!started){
				group.threadStartFailed(this);
			}
		}catch(Throwable ignore){
		}
	}
}
```

run() 源码(基于JDK1.7.0_40)
```java
public void run() {
    if(target != null){
        target.run();
    }
}
```

用start方法来启动线程，真正实现了多线程运行，这时无需等待run方法体代码执行完毕而直接继续执行下面的代码。通过调用Thread类的start()方法来启动一个线程，这时此线程处于就绪（可运行）状态，并没有运行，一旦得到cpu时间片，就开始执行run()方法，这里方法run()称为线程体，它包含了要执行的这个线程的内容，Run方法运行结束，此线程随即终止。start()不能被重复调用。

run()方法只是类的一个普通方法而已，如果直接调用Run方法，程序中依然只有主线程这一个线程，其程序执行路径还是只有一条，还是要顺序执行，还是要等待run方法体执行完毕后才可继续执行下面的代码，这样就没有达到写线程的目的。

总结：调用start方法方可启动线程，而run方法只是thread的一个普通方法调用，还是在主线程里执行。

把需要并行处理的代码放在run()方法中，start()方法启动线程将自动调用run()方法，这是由jvm的内存机制规定的。并且run()方法必须是public访问权限，返回值类型为void。

### sleep()介绍

sleep() 定义在Thread.java中。
sleep() 的作用是让当前线程休眠，即当前线程会从“运行状态”进入到“休眠(阻塞)状态”。sleep()会指定休眠时间，线程休眠的时间会大于/等于该休眠时间；在线程重新被唤醒时，它会由“阻塞状态”变成“就绪状态”，从而等待cpu的调度执行。

# 结果分析

正常情况下，最开始d值为42， `t.start();` 会启动一个用于运行run()方法的新线程，之后直接继续执行下面的代码，无需等待run方法体代码执行完毕，也就是直接继续执行 `doSomething(1000);` ，等待1000毫秒后运行 `420/d`并输出结果；而另一边run函数一旦得到cpu时间片，就开始执行run()方法，也就是说在大于等于1001毫秒后，将会把d赋值为0。也就是理论上，“d赋值为0”会在“420/d”之后执行，所以在正常情况下应该输出10。
也就是这样：
![jpf3.jpg](/images/ST_Lab4JPF/730F61C717A5966E8E1E042E63E438C6.jpg)

但是jpf就是用于并发程序的模型检验，发现数据竞争和死锁等缺陷(defects)。

如果程序的运行过程中，在start()之后的 `doSomething(1000);` 出现了异常，实际的运行时间变得更长，就会使“d赋值为0”优先于“420/d”执行，这样就会出现除以0的异常。
也就是下图所示，在该句加上断点后再执行就会报错。
![jpf4.jpg](/images/ST_Lab4JPF/B175B5737113D134AAA15FABA1085B20.jpg)

也就是说，这是该程序的一个缺陷，在实际操作中会有很大的风险。

# 论文研究

>【篇名】基于符号执行和数据挖掘的路径可达性检测
【作者】范彧
【学位类型】硕士
【授予单位】上海交通大学
【导师】赵建军
【年份】2012
【页数】87


主要内容：基于符号执行和数据挖掘的方法来检测路径的可达性。
文中介绍了限定深度的符号执行算法，并描述了字节码插桩及获取动态运行数据的过程，最后说明了如何从动态数据中挖掘不可达路径。

![jpfpaper.jpg](/images/ST_Lab4JPF/920C902DB9464C3ADE1273C1AE0A81B7.jpg)
上图展示了基于特殊符号执行和数据挖掘方法的大致流程。

首先，对待测的源程序进行限定深度的符号执行。在这种特殊的符号执行下，方法的返回值将作为一种特殊的符号变量处理，称为可疑符号变量，路径条件中带有可疑符号变量的路径称为可疑路径。限定深度的符号执行可以用来获得一部分不可达路径以及一部分可疑路径的信息。随后，利用约束求解器提供的额外信息，一组相关的测试数据被生成。通过大量地重复执行插桩过的程序，动态的路径信息被获得。这些信息包含了程序在不同输入下的执行路径。利用关联规则挖掘技术可以从动态路径信息中挖掘出路径条件中的相互关联，从而最终从可疑路径中找出真正不可达的。


## 知识点
### 代码插桩
通过在程序代码中插入额外的语句来**监视程序中感兴趣的变量的状态**，可以帮助程序员更好地找到软件中的缺陷。
具体为：在保持源程序逻辑不变的情况下**插入探针**，通过执行探针来抛出源程序的**特征数据**。通过分析这些数据，可以获得程序的控制流和数据流信息，对程序进行进一步的分析。

### 符号执行
符号执行的主要思想是使用**符号值**代替实际数据作为程序的**输入**，使用**符号表达式**来表示**程序变量**。因此，程序执行的输出是一个**关于输入符号值的函数**。这里的符号值是一个**固定**的值，只是不知道取值是多少。这需要和程序中的变量严格区分开来，后者是会在程序执行过程中不断变化的。符号执行是普通程序执行的一种自然拓展，使用这种方式可以系统地遍历程序所有可能的执行路径。值得一提的是，这里的输入是指任何来自方法外部的数据，并不限于方法形参，同时还包括全局变量等。

```C
int x, y;
L1: if (x > y) {
L2:    x = x + y;
L3:    y = x ‐ y;
L4:    x = x ‐ y;
L5:    if (x ‐ y > 0)
L6:      assert (false);
L7: }
```
是当 x > y 时将两者交换， x <= y 时什么都不做。

下图为符号执行树：
![jpfpaper1.jpg](/images/ST_Lab4JPF/64907401A44CD77A28091436929124D3.jpg)

- IP 是一个**指令指针**（Instruction Pointer），指向下一条将要执行的指令。
- PC **路径条件**（Path Condition）是由输入的符号变量组成的布尔表达式，用来表示使得程序沿着这条路径执行的**约束条件**。
- V 是一组有序对的集合。每一组有序对表示一个**程序变量到符号表达式的映射**，记作<variable, expression>。

因此，符号状态记作SymbolicState=\<IP,PC,V\> 。

其中带阴影的部分，路径条件不存在，所以这一步骤只输出一个新的符号状态S7


Symbolic PathFinder集成了符号执行和模型检测，可以用来完成代码的检测以及自动化测试用例生成。利用 Symbolic PathFinder 可以生成高覆盖率的结构化测试代码。
与一般的符号检测工具不同，Symbolic PathFinder并不依赖代码插桩，而是利用JPF的拓展接口实现了非标准的字节码解释，在执行被测代码的同时系统地生成和执行符号执行树。



# JPF的符号执行（根据论文进行拓展）
参考：
[http://babelfish.arc.nasa.gov/trac/jpf/wiki/projects/jpf-symbc](http://babelfish.arc.nasa.gov/trac/jpf/wiki/projects/jpf-symbc)
[http://babelfish.arc.nasa.gov/trac/jpf/wiki/projects/jpf-symbc/doc](http://babelfish.arc.nasa.gov/trac/jpf/wiki/projects/jpf-symbc/doc)



## 准备
### jpf-symbc
使用Mercuial下载jpf-symbc的仓库，执行下列命令
```
$ hg clone http://babelfish.arc.nasa.gov/hg/jpf/jpf-symbc
$ cd jpf-symbc
$ ant test
```
![jpfs.jpg](/images/ST_Lab4JPF/7F5606082735DCEED5284797371BBB3F.jpg)
### 配置site.properties文件
在家目录下新建.jpf文件夹，在该目录下新建site.properties文件

```
# can only expand system properties
jpf-core = ${user.home}/jpf-core

# annotation properties extension
jpf-aprop = ${jpf.home}/jpf-aprop
extensions+=,${jpf-aprop}

# numeric extension
jpf-numeric = ${jpf.home}/jpf-numeric
extensions+=,${jpf-numeric}

# symbolic extension
jpf-symbc = ${jpf.home}/jpf-symbc
extensions+=,${jpf-symbc}

# concurrent extension
#jpf-concurrent = ${jpf.home}/jpf-concurrent
#extensions+=,${jpf-concurrent}

jpf-shell = ${jpf.home}/jpf-shell
extensions+=,${jpf-shell}

jpf-awt = ${jpf.home}/jpf-awt
extensions+=,${jpf-awt}

jpf-awt-shell = ${jpf.home}/jpf-awt-shels
extensions+=,${jpf-awt-shell}

```
## 编译并运行
成功后在jpf-symbc文件夹下新建srcExample文件夹，新建java文件和jpf文件：
MyClass.java
```java
public class MyClass {
	public int myMethod(int x, int y){
		int z = x + y;
		if (z > 0) {
			if(y>0){
				z = 1;
			} else{
				z = -1;
			}
			System.out.println("path 1 explored");
		} else {
			if(x>0){
				z = z - x;
			} else{
				z = z + x;
			}
			System.out.println("path 2 explored");
		}			
		z = 2 * z;
		return z;
	}
																											
	public static void main(String[] args){
		MyClass mc = new MyClass();
		mc.myMethod(1, 2);
	}
}
```

MyClass.jpf
```java
target=MyClass
classpath=${jpf-symbc}/buildExample
sourcepath=${jpf-symbc}/srcExample
symbolic.method=MyClass.myMethod(sym#sym)
```


要debug运行得到MyClass.class: `javac -g MyClass.java`

在jpf-symbc文件夹下新建buildExample文件夹，将MyClass.class移动到文件夹内，运行后可以看到输出：

![jpfs11.jpg](/images/ST_Lab4JPF/A7157371BDE9D70DB8E02464F4CA32B2.jpg)

然后返回srcExample文件夹下，执行  `~/jpf-core/bin/jpf MyClass.jpf`
得到下图所示结果：

![jpfs12.jpg](/images/ST_Lab4JPF/08BEA7AD0120B60299F6F4B900B9607B.jpg)


同理论文中的程序也进行一次操作:

```java
public class MyClass1{																								
	public void myMethod(int x, int y){
		if (x > y) {
			x = x + y;
			y = x - y;
			x = x - y;
			if (x - y > 0)
				assert(false);
		}
	}
	public static void main(String[] args){
		MyClass1 mc1 = new MyClass1();
		mc1.myMethod(1, 2);
	}
}
```


执行结果如图所示：

![jpfs21.jpg](/images/ST_Lab4JPF/A6F47596B5198CA8D3841F1A08DDCC9A.jpg)


# 出现的问题
## CentOS虚拟机无法联网

![jpfqq1.jpg](/images/ST_Lab4JPF/D4C208B49778E12A4F5E1C5F8788ABD1.jpg)
解决方法：
参考后面的网站进行操作，修改Centos7中ifcfg-xxx文件。需要修改添加的有以下几行，这个是例子：

```
BOOTPROTO=static  #设置静态Ip
ONBOOT=yes  #这里如果为no的话就改为yes，表示网卡设备自动启动
GATEWAY=192.168.10.155  #这里的网关地址就是mac下cat /Library/Preferences/VMware Fusion/vmnet8/nat.conf中#NAT gateway address这一行下面的ip地址
IPADDR=192.168.10.150  #配置ip，在上一步已经设GATEWAY处于192.168.10.xxx这个范围，我就随便设为150了，只要不和网关相同均可
NETMASK=255.255.255.0#子网掩码
DNS1=202.96.128.86#dns服务器1，填写你所在的网络可用的dns服务器地址即可
DNS2=223.5.5.5#dns服器2
```

最后执行 `service network restart` ，ping下百度的域名
![jpfqq2.jpg](/images/ST_Lab4JPF/E476F31B88F2D70F83D140B68764FB8C.jpg)
参考：
https://blog.csdn.net/qiuqurenkong/article/details/78598650
https://blog.csdn.net/a785975139/article/details/53023590


## jpf-symbc 报错 [SEVERE] cannot load application class 某class
错误原因：
jpf中需要设置两个路径，classpath用来放置生成了所有调试信息的class文件，sourcepath用来放置Java文件和jpf文件。
因此需要`javac -g`来执行，并且把这三个文件放到相应的位置，并在jpf中配置好路径。


# 参考文献
[1]范彧.基于符号执行和数据挖掘的路径可达性检测[D].上海交通大学,2012.

