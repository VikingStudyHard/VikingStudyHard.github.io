---

title: 【软件测试上机】1 在IDEA上使用JUnit和Eclemma进行测试

date: 2018-3-22

categories: [软件测试, 本科课程]

tags: 

 - SoftwareTest
 - 软件测试上机

description: 在IDEA上使用JUnit对判断三角形的函数进行测试

---

### 下载junit和hamcrest all
junit 下载地址：
http://mvnrepository.com/artifact/junit/junit
hamcrest all下载地址：
http://mvnrepository.com/artifact/org.hamcrest/hamcrest-all

得到
junit-4.12.jar和hamcrest-all-1.3.jar

### 打开IDEA
1.新建项目，建立与src平级的文件夹：
* lib文件夹，用于导入jar包
* test文件夹，作为测试文件夹


![IMAGE](/images/ST_Lab1JUnit-IDEA/B1DE6A8DC267E313259029C2935DDE98.jpg)

2.将junit-4.12.jar和hamcrest-all-1.3.jar放入lib文件夹中

![IMAGE](/images/ST_Lab1JUnit-IDEA/4F768A9E99FCAB3C8300F5B742F8F6D1.jpg)

3.将jar包导入IDEA

File -> Project Structure -> Modules

![IMAGE](/images/ST_Lab1JUnit-IDEA/D2A1502212F9CDF31065C3D33C329E2A.jpg)

Dependencies —> 左下方+号 —> JARs or Directories

![IMAGE](/images/ST_Lab1JUnit-IDEA/C7547C7C00382E0392B1CCDC74EE7F04.jpg)

选择JAR包，apply

![IMAGE](/images/ST_Lab1JUnit-IDEA/A0508FA9E6E63AAE7AD0D41606085FF9.jpg)

4.将test文件夹设为Test Source Root

test文件夹 -> 右键 -> Mark Directory as -> Test Sources Root



test文件夹图标发生改变：

![IMAGE](/images/ST_Lab1JUnit-IDEA/B3745DA55B613D075917A33EDDA66720.jpg)

5.在src文件夹中新建class：Main.java
要求：Function triangle takes three integers a,b,c which are length of triangle sides; calculates whether the triangle is equilateral, isosceles, or scalene. 


```java
public class Main {
    public static String testTriangle(int a,int b,int c){
        if(a<0 || b<0 || c<0){
            return "illegal triangle";
        }else if(a+b>c && a+c>b && b+c>a){
            if(a==b && a==c){
                return "equilateral triangle";
            }else if(a==b || b==c || c==a){
                return "isosceles triangle";
            }else{
                return "scalene triangle";
            }
        
            return "illegal triangle";
        }
    }
}
```
6.Navigate -> Test -> Create New Test

![IMAGE](/images/ST_Lab1JUnit-IDEA/71AE5DAE996134530F883C070479DD15.jpg)


![IMAGE](/images/ST_Lab1JUnit-IDEA/4C01F1383F9080DDEE8576C4A5B75E59.jpg)

![IMAGE](/images/ST_Lab1JUnit-IDEA/FD951535174474C81C9D98E229F4C542.jpg)

在test文件夹下得到MainTest.java：

![IMAGE](/images/ST_Lab1JUnit-IDEA/BBE421C8BA022D86DC72A4EA7E523297.jpg)

编写测试内容
```java
import org.junit.Test;

import static org.junit.Assert.*;

public class MainTest {

    @Test
    public void testTriangle() throws Exception {
        int a,b,c;
        a = 1;
        b = 2;
        c = 3;
        assertEquals(("illegal triangle"),Main.testTriangle(a,b,c));

        a = 1;
        b = 1;
        c = 1;
        assertEquals(("equilateral triangle"),Main.testTriangle(a,b,c));

        a = 1;
        b = 1;
        c = -1;
        assertEquals(("illegal triangle"),Main.testTriangle(a,b,c));

        a = 3;
        b = 3;
        c = 5;
        assertEquals(("isosceles triangle"),Main.testTriangle(a,b,c));

        a = 3;
        b = 4;
        c = 5;
        assertEquals(("scalene triangle"),Main.testTriangle(a,b,c));
    }

}
```

![IMAGE](/images/ST_Lab1JUnit-IDEA/8B5E3321DB60F179EDCB64AB20920148.jpg)
运行起来：

![IMAGE](/images/ST_Lab1JUnit-IDEA/77A4730C8630658F97032D3CAB062D99.jpg)


如果把上面代码改成错误的：

```java
        a = 2;
        b = 2;
        c = 3;
        assertEquals(("illegal triangle"),Main.testTriangle(a,b,c));
```

就会得到：

![IMAGE](/images/ST_Lab1JUnit-IDEA/A2A23CD04066A6B8A6A71223EFB2F5EA.jpg)


### 进行覆盖测试

EclEmma是一个开源的软件测试工具，可以在编码过程中查看代码调用情况、也可以检测单覆盖率。它是Eclipse的插件。

#### 编辑测试设置
通过Run → Edit Configurations或工具栏上的标签来调整测试运行配置。

![IMAGE](/images/ST_Lab1JUnit-IDEA/382536A0172275DA93900F5849876B92.jpg)



在Configuration选项卡选择需要运行的测试。比如，可以从一个类、程序包、测试套件或甚至模式中运行所有的测试。这里的Fork模式让用户在一个单独的进程运行每个测试。


![IMAGE](/images/ST_Lab1JUnit-IDEA/DF85FEBC2D76CD4A0AD3B1F191317BAE.jpg)


可以调整覆盖率设置。目前IntelliJ IDEA支持3种测量覆盖率引擎。默认情况下它使用自己的引擎，也可以选择JaCoCo、Emma引擎。还可以在这里选择覆盖率模式。

![IMAGE](/images/ST_Lab1JUnit-IDEA/C92E5164F2AFB757AB8D1EED1A37A91E.jpg)


#### 运行覆盖

收集覆盖率, 通过Run → Run 'MainTest.testTriangle' with Coverage或工具栏上的选项运行特定模式的测试。

![IMAGE](/images/ST_Lab1JUnit-IDEA/FC9CE42B76C7F1B8077291AE184F1934.jpg)

例如：
##### 真实值和测试值相同：
![IMAGE](/images/ST_Lab1JUnit-IDEA/4B7AC488A03DD00A8A531A1B03F97F57.jpg)

![IMAGE](/images/ST_Lab1JUnit-IDEA/DB54406F4FEC24DB5C458B5F18108561.jpg)


##### 真实值和测试值不同：
![IMAGE](/images/ST_Lab1JUnit-IDEA/7A8A9A74CFFDAEA716B660184DF13B7F.jpg)

![IMAGE](/images/ST_Lab1JUnit-IDEA/67BF0939ABB35FEA40F4AC6572E6027C.jpg)
