---

title: 【软件测试上机】2 Selenium

date: 2018-4-13

categories: [软件测试, 本科课程]

tags: 

 - SoftwareTest
 - 软件测试上机

description: 安装SeleniumIDE插件,使用SeleniumIDE录制脚本和导出脚本,编写Selenium Java WebDriver程序

---


# 安装SeleniumIDE插件
这里，我试用两个浏览器进行的实验
## Firefox
1.下载Firefox浏览器
Firefox全历史版本下载： http://ftp.mozilla.org/pub/mozilla.org//firefox/releases/ 
2.若是56.0以上的版本，可直接用Firefox打开 https://addons.mozilla.org/en-US/firefox/addon/selenium-ide/ ，点击“Add to Firefox”.其他的需要在浏览器“添加组件”，然后搜索selenium ide 选择历史版本中匹配的版本。

注：最新（59.0）版本**没有导出脚本功能**
## Chrome
应用->应用商店->搜selenuim->添加Katalon Recorder 

# 使用SeleniumIDE录制脚本和导出脚本
这里，我试用一下三种方法进行实验：

## Firefox 59.0版本（没有导出脚本功能）
![seleniumIDE.jpg](/images/ST_Lab2Selenium/F8372ACAAF0DB82B9C59C0A4478AFD1B.jpg)
### 录制脚本
1.点击红色圆形按钮，开始录制
selenium ide 右上角的在红色圆表示录制结束或尚未录制，红色正方形的时候表示正在录制。
2.访问 https://psych.liebes.top/st ，输入学号和默认密码，点击提交，得到git地址，访问该git地址
3.点击红色正方形按钮，结束录制。将项目名、测试名重命名。
![seleniumIDE1.jpg](/images/ST_Lab2Selenium/4F4202FB3B0945A6039CD3E66C5658D0.jpg)
4.点击运行按钮即可进行自动测试
### 脚本操作
1.添加一条脚本：点击右键，出现 Insert New Command, 在下方编辑
2.添加注释：点击右键， 出现 Insert New Comment，点击之后在command框中添加注释如搜索

## Firefox 42.0版本
安装的组件有：
Selenium IDE 2.9.1.1-signed
Selenium IDE Button 1.2.0.1-signed.1-signed
![seleniumIDE5.jpg](/images/ST_Lab2Selenium/2008A1B9D38DD8E3264F33A9B35A08E3.jpg)
### 录制脚本
1.默认点开组件按钮自动开始录制
selenium ide 右上角的在红色实心圆表示录制结束或尚未录制，红色空心圆的时候表示正在录制。
2.访问 [https://psych.liebes.top/st](https://psych.liebes.top/st)，输入学号和默认密码，点击提交，得到git地址，访问该git地址
3.点击红色空心圆形按钮，结束录制。将项目名、测试名重命名。
![seleniumIDE6.jpg](/images/ST_Lab2Selenium/B156FCE47DBC92E1DD012AF18F8E65E3.jpg)
4.点击运行按钮即可进行自动测试
![seleniumIDE8.jpg](/images/ST_Lab2Selenium/8ED2D43C90439EAB495867E61E0011DC.jpg)
###导出脚本
![seleniumIDE7.jpg](/images/ST_Lab2Selenium/7C21D487725B6ABAF307B8523C9299D3.jpg)
得到
```java
package com.example.tests;

import java.util.regex.Pattern;
import java.util.concurrent.TimeUnit;
import org.junit.*;
import static org.junit.Assert.*;
import static org.hamcrest.CoreMatchers.*;
import org.openqa.selenium.*;
import org.openqa.selenium.firefox.FirefoxDriver;
import org.openqa.selenium.support.ui.Select;

public class Lab2 {
  private WebDriver driver;
  private String baseUrl;
  private boolean acceptNextAlert = true;
  private StringBuffer verificationErrors = new StringBuffer();

  @Before
  public void setUp() throws Exception {
    driver = new FirefoxDriver();
    baseUrl = "https://psych.liebes.top/";
    driver.manage().timeouts().implicitlyWait(30, TimeUnit.SECONDS);
  }

  @Test
  public void testLab2() throws Exception {
    driver.get(baseUrl + "/st");
    driver.findElement(By.id("username")).clear();
    driver.findElement(By.id("username")).sendKeys("3015218155");
    driver.findElement(By.id("password")).clear();
    driver.findElement(By.id("password")).sendKeys("218155");
    driver.findElement(By.id("submitButton")).click();
    driver.findElement(By.cssSelector("p.login-box-msg")).click();
  }

  @After
  public void tearDown() throws Exception {
    driver.quit();
    String verificationErrorString = verificationErrors.toString();
    if (!"".equals(verificationErrorString)) {
      fail(verificationErrorString);
    }
  }

  private boolean isElementPresent(By by) {
    try {
      driver.findElement(by);
      return true;
    } catch (NoSuchElementException e) {
      return false;
    }
  }

  private boolean isAlertPresent() {
    try {
      driver.switchTo().alert();
      return true;
    } catch (NoAlertPresentException e) {
      return false;
    }
  }

  private String closeAlertAndGetItsText() {
    try {
      Alert alert = driver.switchTo().alert();
      String alertText = alert.getText();
      if (acceptNextAlert) {
        alert.accept();
      } else {
        alert.dismiss();
      }
      return alertText;
    } finally {
      acceptNextAlert = true;
    }
  }
}

```
## Chrome的Katalon Automation版本
![seleniumIDE2.jpg](/images/ST_Lab2Selenium/BB112B076CADCA62D1D41304E9A01DA0.jpg)
### 录制脚本
1.点击Record，开始录制
2.访问 [https://psych.liebes.top/st](https://psych.liebes.top/st) ，输入学号和默认密码，点击提交，得到git地址，访问该git地址
3.点击Stop，结束录制
![seleniumIDE3.jpg](/images/ST_Lab2Selenium/C81D13C6E4BA57DB43D71EA62EF90528.jpg)
4.点击运行按钮即可进行自动测试
###导出脚本
![seleniumIDE4.jpg](/images/ST_Lab2Selenium/1BB4F0BEC5D390A3B08535A19171FF4A.jpg)
得到
```java
package com.example.tests;

import java.util.regex.Pattern;
import java.util.concurrent.TimeUnit;
import org.junit.*;
import static org.junit.Assert.*;
import static org.hamcrest.CoreMatchers.*;
import org.openqa.selenium.*;
import org.openqa.selenium.firefox.FirefoxDriver;
import org.openqa.selenium.support.ui.Select;

public class UntitledTestCase {
  private WebDriver driver;
  private String baseUrl;
  private boolean acceptNextAlert = true;
  private StringBuffer verificationErrors = new StringBuffer();

  @Before
  public void setUp() throws Exception {
    driver = new FirefoxDriver();
    baseUrl = "https://www.katalon.com/";
    driver.manage().timeouts().implicitlyWait(30, TimeUnit.SECONDS);
  }

  @Test
  public void testUntitledTestCase() throws Exception {
    driver.get("https://psych.liebes.top/st");
    driver.findElement(By.id("username")).click();
    driver.findElement(By.id("username")).clear();
    driver.findElement(By.id("username")).sendKeys("3015218155");
    driver.findElement(By.id("password")).click();
    driver.findElement(By.id("password")).clear();
    driver.findElement(By.id("password")).sendKeys("218155");
    driver.findElement(By.id("submitButton")).click();
    driver.findElement(By.xpath("//p")).click();
  }

  @After
  public void tearDown() throws Exception {
    driver.quit();
    String verificationErrorString = verificationErrors.toString();
    if (!"".equals(verificationErrorString)) {
      fail(verificationErrorString);
    }
  }

  private boolean isElementPresent(By by) {
    try {
      driver.findElement(by);
      return true;
    } catch (NoSuchElementException e) {
      return false;
    }
  }

  private boolean isAlertPresent() {
    try {
      driver.switchTo().alert();
      return true;
    } catch (NoAlertPresentException e) {
      return false;
    }
  }

  private String closeAlertAndGetItsText() {
    try {
      Alert alert = driver.switchTo().alert();
      String alertText = alert.getText();
      if (acceptNextAlert) {
        alert.accept();
      } else {
        alert.dismiss();
      }
      return alertText;
    } finally {
      acceptNextAlert = true;
    }
  }
}
```
# 编写Selenium Java WebDriver程序
测试input.xlsx表格中的学号和git地址的对应关系是否正确。
## 步骤
1.新建项目，新建文件夹lib，导入junit-4.12.jar，hamcrest-all-1.3.jar和selenium-java-2.53.1.jar
2.File-> Project Structure -> Modules -> Dependencies —> 左下方+号 —> JARs or Directories 导入以上jar包
3.新建文件夹resource，设置为Resources Root，把input.xlsx另存为input.csv放在里面。
4.打开src/Main.Java

## 代码
```java
package com.company;
import java.io.IOException;
import java.nio.charset.Charset;
import org.openqa.selenium.By;
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.*;
import org.openqa.selenium.firefox.FirefoxDriver;

import com.csvreader.CsvReader;
import java.util.concurrent.TimeUnit;

public class Main{
    public static void main(String[] args){
        System.out.println("未通过列表：");
        int pass = 0;
        int unpass = 0;
        String BaseURL ="https://psych.liebes.top";
        long startTime = System.currentTimeMillis(); //程序开始记录时间
        try {
            CsvReader r = new CsvReader("/Users/viking/Documents/IDEAproject/Lab2/resource/input.csv", ',', Charset.forName("UTF-8"));
            while (r.readRecord()) {
                //读取CSV文件中数据
                String str = r.getRawRecord();
                String[] strList = str.split(",");
                String number = strList[0].trim();
                if(strList.length == 2) {
                    String address = strList[1].trim();
                    int x = address.length();
//                    if (address.charAt(x - 1) == '/') {
//                        address = strList[1].trim().substring(0, x - 1);
//                    } else {
                        address = strList[1].trim();
//                    }

                    String pwd = number.substring(number.length() - 6, number.length());

                    //对应@Before
                    WebDriver driver = new FirefoxDriver();
                    driver.manage().timeouts().implicitlyWait(30, TimeUnit.SECONDS);

                    //对应@Test
                    driver.get(BaseURL + "/st");
                    driver.findElement(By.id("username")).clear();
                    driver.findElement(By.id("username")).sendKeys(number);
                    driver.findElement(By.id("password")).clear();
                    driver.findElement(By.id("password")).sendKeys(pwd);
                    driver.findElement(By.id("submitButton")).click();

                    String targetGit = driver.findElement(By.cssSelector("p.login-box-msg")).getAttribute("innerHTML").trim();
                    if (address.equals(targetGit))
                        pass++;
                    else{
                        unpass++;
                        System.out.println(number + " 未通过：git不匹配。");
                        System.out.println("网站反馈的是：" + targetGit);
                        System.out.println("表格给出的是：" + address);
                    }
                    driver.close();
                }else{
                    unpass++;
                    System.out.println(number + " 未通过：表格中未读取到git");
                }
            }
            r.close();
            long endTime = System.currentTimeMillis(); //程序结束记录时间
            long TotalTime = endTime - startTime; //总消耗时间
            System.out.println("--------测试结束--------");
            System.out.println("通过"+pass+"人，未通过"+unpass+"人。");
            System.out.println("测试总时长："+TotalTime);
        }catch(IOException e){
            e.printStackTrace();
        }

    }
}

```

## 出现的问题

1.xlsx转化成csv出现格式错误，如多余的换行，已手动去掉。
2.对于`WebDriver driver = new FirefoxDriver();`

出现报错

```java
Exception in thread "main" java.lang.NoClassDefFoundError: com/google/common
```
解决方案：
导入selenium-server-standalone-2.53.0.jar
3.csv用UTF-8编码，表头出现问题。第一行输入的学号之前会有一串隐藏的字符，因此在输入时会出现账号错误，要求重新输入的提示，因此并不能匹配到目标git。
问题原因：
UTF-8 编码的文件可以分为no BOM 和 BOM两种格式。
有bom头的存储或者字节流，它一定是unicode字符集编码。到底属于那一种（utf-8还是utf-16或是utf-32)，通过头可以判断出来。
在utf-8编码文件中BOM在文件头部，占用三个字节，用来标示该文件属于utf-8编码。UTF-8的BOM是 EFBBBF，因为UE载入UTF-8文件会转成Utf16，上述的EFBBBF 在Utf16中是FFFE（Unicode-LE的BOM）。
解决方案：
用sublime打开文件后，另存为选项的编码格式里，选择utf-8 (无bom头）。
4.在git部分，有前面带空格的需要用trim()进行去除。
5.在git部分，csv中有以‘/’结尾的网址，事实上和网站给出的没有‘/’的网址是一个网址。因此读取csv时需要判断git地址是否以‘/’结尾，如果是，则去掉后再与网站上的地址进行对比。
但是后来到3015207537时发现，网站上给出的就是结尾带/的网址，所以如果事先进行了处理，反而不对了。
所以最后决定：不进行处理了。
因此，试验后发现，网站基本上和表格里的网址匹配，未通过的都是因为结尾‘/’有无。
6.对于3015218150，表格里没有对应的git地址，也就是没有strList[1]，因此在前面需要对strList的长度进行判断，否则会报错。
7.最后添加了已通过、未通过的人数统计，并显示了测试时间。


## 结果
结果图片已经过处理
![seleniumIDE9.jpg](/images/ST_Lab2Selenium/F1E180DBBDC5034741CB92D8392FBFDB.jpg)
可见有6个未通过的，其中5个是因为git网址结尾的/，还有1个是没有对应的git地址
运行时间约为576s