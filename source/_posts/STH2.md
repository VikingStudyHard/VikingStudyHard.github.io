---

title: 【软件测试】02 设计测试用例

date: 2018-03-10

categories: [软件测试, 本科课程]

tags: 

 - SoftwareTest

description: Below are two faulty programs. Each includes a test case that results in failure.

---
# TIPS
## Software Faults, Errors & Failures
Software **Fault** : A static defect in the software
Faults in software are equivalent to design mistakes in hardware
错误的根本原因，指程序中静态的缺陷，也就是指在程序中存在的编程错误。

Software **Failure** : External, incorrect behavior with respect to the requirements or other description of the expected behavior
与需求或期望的行为的描述有关的、外部的、不正确的行为，指程序产生与我们期望不同的结果

Software **Error** : An incorrect internal state that is the manifestation of some fault
不正确的内部状态
## 找出一个失败的三个条件：
程序中包含错误的位置必须找到(可达性，reachability)
执行该位置后，程序的状态必须是不正确的(影响,infection)
受到影响的状态必须传播出来，引起该程序某个输出是不正确的(传播，propagation)

  
# QUESTIONS 
## Two faulty programs
Below are two faulty programs. Each includes a test case that results in failure. Answer the following questions (in the next slide) about each program.

```java
public int findLast (int[] x, int y){ 
//Effects: If x==null throw NullPointerException
// else return the index of the last element 
// in x that equals y.
// If no such element exists, return -1
  for (int i=x.length-1; i > 0; i--)
  {
    if (x[i] == y)
    {
      return i; 
    }
  }
  return -1;
}
// test: x=[2, 3, 5]; y = 2 
// Expected = 0
```


```java
public static int lastZero (int[] x){ 
//Effects: if x==null throw NullPointerException
// else return the index of the LAST 0 in x. 
// Return -1 if 0 does not occur in x
  for (int i = 0; i < x.length; i++)
  {
    if (x[i] == 0)
    {
      return i;
    }
  } 
  return -1; 
}
// test: x=[0, 1, 0] 
// Expected = 2
```

## Specific issues
* Identify the fault.
* If possible, identify a test case that does not execute the fault.(Reachability)
* If possible, identify a test case that executes the fault, but does not result in an error state.
* If possible identify a test case that results in an error, but not a failure.

# MY ANSWERS
## 程序一
1. dentify the fault.
程序一是要找到某个数在数组中最后一次出现的位置，如果该数字并没有出现，那么返回-1。
for循环为：
```java
for (int i=x.length-1; i > 0; i--)
```

指的是i从长度减去1开始递减到1结束。而遍历全体数组应该从x[x.length-1]到x[0]，所以应该把上面的句子改为：

```java
for (int i=x.length-1; i > -1; i--)
```
fault在于遍历到`x[1]`停止，没有遍历到x[0]。

2.If possible, identify a test case that does not execute the fault.(Reachability)
这个测试用例要求不到达故障的代码。
当x为空，即null，在初始化i值时就抛出空指针异常，因此不会执行后面的代码。   

3.If possible, identify a test case that executes the fault, but does not result in an error state.
这个测试用例要求到达出错位置，但不能得到内部错误error。
因为fault时是静态的，如果执行了fault就一定会产生错误的内部状态error，所以不存在该用例。

4.If possible identify a test case that results in an error, but not a failure.
这个测试用例要求执行错误代码，产生错误的内部状态error，但不能得到错误外部状态failure。
当x为{1,3,4}，y为2时，i最开始时等于2，此时x[2]为4，与y不相等；i 等于1时，x[1]为3，与y不相等，跳出for循环，返回值赋值为-1，程序结束。此时执行到了x[1]，到达了出错位置，但是由于结果和期望值相等，所以没有导致外部错误failure。

## 程序二
1.Identify the fault.
程序二是要找到0在数组中最后一次出现的位置，如果数字0并没有出现，那么返回-1。
for循环为：
```java
for (int i = 0; i < x.length; i++)
```
指的是i从0开始递增到x的长度减1结束。而想要找到最后一次出现的位置，应该让i从x的长度减1到0递减，所以应该把上面的句子改为：

```java
for (int i=x.length-1; i > -1; i--)
```
fault在于遍历反了。

2.If possible, identify a test case that does not execute the fault.(Reachability)
这个测试用例要求不到达故障的代码。
当x为空，即null，在初始化i值时就抛出空指针异常，因此不会执行后面的代码。   

3.If possible, identify a test case that executes the fault, but does not result in an error state.
这个测试用例要求到达出错位置，但不能得到内部错误error。
因为fault时是静态的，如果执行了fault就一定会产生错误的内部状态error，所以不存在该用例。

4.If possible identify a test case that results in an error, but not a failure.
这个测试用例要求执行错误代码，产生错误的内部状态error，但不能得到错误外部状态failure。
当x为{0,1,2}，到达了出错位置产生了错误的内部状态error，但是由于结果和期望值相等，所以没有导致外部错误failure。

# MY DOUBTS
就程序一来说：
1.关于Reachability 可达性。没有到达指的是没有执行到`x[1]`，还是没有执行到for循环就停止了？`x=[2,3,5],y=5`，算“没有Reachability”吗？
2.存在fault，执行到达了fault就会产生内部错误error？（所以我觉得第三问的结果都是“不存在”）
3.报错属于错误的外部状failure（没有出结果）还是内部状态error？

# REPLY
3. 报错、异常属于内部错误error
待补充

