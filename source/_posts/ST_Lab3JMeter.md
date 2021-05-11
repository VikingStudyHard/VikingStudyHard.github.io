---

title: 【软件测试上机】3 JMeter压力测试

date: 2018.04.18

categories: [软件测试, 本科课程]

tags: 

 - SoftwareTest
 - 软件测试上机

description: 基于ECShop进行Jmeter压力测试

---
# 实验目标

安装虚拟机，并安装一套LAMP（Linux+Apache+Mysql+PHP）待测系统，推荐[ECShop](http://www.ecshop.com)，基于此进行Jmeter压力测试，并在测试后得出Jmeter测试报告，并根据sysstat得出Linux服务器的CIMN（CPU、IO、Memory以及Network）的性能。

# 准备工作
## 看教程视频

## 下载 CentOS

[Download CentOS](https://www.centos.org/download/)
主界面上有三个不同的镜像文件类型的CentOS，分别为DVD ISO、Everything ISO、Minimal ISO。
* DVD ISO：此镜像类型为普通光盘安装版，可离线安装到计算机硬盘上，包含大量的常用软件，一般选择这种镜像类型即可。
* Everything ISO：这个镜像涵盖了上种镜像的内容，并对其进行补充，集成了所有软件。
* Minimal ISO：这个版本为精简版的镜像，可以安装一个基本的CentOS系统，包含了可启动系统基本所需的最小安装包。

选择DVD ISO，得到 CentOS-7-x86_64-DVD-1708.iso (4.52G)

## 安装 CentOS 7 虚拟机
VMware Fusion 专业版 10.0.1
文件 -> 新建 -> 拖入下载的IOS文件 -> ... -> 储存
出现黑色的界面，安装开始。
选择语言、在网络和主机名中打开以太网、选择输入法、在软件选择中，若选择“最小安装”，即命令行界面；若想有界面则选择“GNOME桌面”、选择安装位置、最后选择安装。
设置root密码和创建一个用户，开始安装。

重启登录系统。（按提示往下做就好了）


## 安装 MySQl

1.安装
CentOS担心使用MySQL会引来版权问题，所以改为集成MariaDB，所以`yum install mysql`安装的是mariadb-5.5.56-2.el7.x86_64，我们要下载并安装MySQL官方的 Yum Repository

`wget -i -c http://dev.mysql.com/get/mysql57-community-release-el7-10.noarch.rpm`直接下载安装用的Yum Repository，大概25KB的样子，然后就可以直接yum安装了。

`yum -y install mysql57-community-release-el7-10.noarch.rpm`之后就开始安装MySQL服务器。

`yum -y install mysql-community-server` 安装完成后就会覆盖掉之前的mariadb。

2.启动MySQL服务 `systemctl start  mysqld.service`

3.查看MySQL运行状态 `systemctl status mysqld.service` 
运行状态如图：
![test6.jpg](/images/ST_Lab3JMeter/8AF9604ADD3B9F5BC343C0281A24E82D.jpg)

4.此时MySQL已经开始正常运行，不过要想进入MySQL还得先找出此时root用户的密码，通过如下命令可以在日志文件中找出密码 `grep "password" /var/log/mysqld.log`
得到
`2018-04-19T03:33:31.430509Z 1 [Note] A temporary password is generated for root@localhost: %/XfesdJl7=j`

5.如下命令进入数据库：
`mysql -uroot -p`连接MYSQL 格式为`mysql -h主机地址 -u用户名 -p用户密码`
输入初始密码，此时不能做任何事情，因为MySQL默认必须修改密码之后才能操作数据库
`mysql> set password for 用户名@localhost = password('新密码');  `
一般设置后会出现`ERROR 1819 (HY000): Your password does not satisfy the current policy requirements`
需要
```
mysql> set global validate_password_policy=0;
mysql> set global validate_password_length=1;
```
再设置`mysql> set password for 用户名@localhost = password('新密码'); `


6.常用命令
* 显示当前数据库服务器中的数据库列表：
　　`mysql> SHOW DATABASES;`
* 建立数据库：
　　`mysql> CREATE DATABASE 库名;`
* 建立数据表：
　　`mysql> USE 库名;`
　　`mysql> CREATE TABLE 表名 (字段名 VARCHAR(20), 字段名 CHAR(1));`
* 删除数据库：
　　`mysql> DROP DATABASE 库名;`
* 删除数据表：
　　`mysql> DROP TABLE 表名;`
* 将表中记录清空：
　　`mysql> DELETE FROM 表名;`
* 往表中插入记录：
　　`mysql> INSERT INTO 表名 VALUES ("hyq","M");`
* 更新表中数据：
 　`mysql-> UPDATE 表名 SET 字段名1='a',字段名2='b' WHERE 字段名3='c';`
* 用文本方式将数据装入数据表中：
　　`mysql> LOAD DATA LOCAL INFILE "D:/mysql.txt" INTO TABLE 表名;`
* 导入.sql文件命令：
　　`mysql> USE 数据库名;`
　　`mysql> SOURCE d:/mysql.sql;`

7.进入mysql后，新建数据库
`mysql> CREATE DATABASE ecshop;`

## 安装 php
`yum install php`

编辑配置文件`vim /etc/php.ini`
`;date.timezone =`把前面的分号去掉，改为`date.timezone = PRC`

安装php的扩展
`yum install php-mysql php-gd php-imap php-ldap php-odbc php-pear php-xml php-xmlrpc`


## 性能测试相关工具
### top 监控每一个进程的资源占用
桌面右键“打开终端”，输入`top`
![test1.jpg](/images/ST_Lab3JMeter/749789654249E42B6152849C2599C5F7.jpg)

### sysstat 统计系统的各种资源的占用情况
1.在终端输入`yum list sysstat`
![test2.jpg](/images/ST_Lab3JMeter/A730CBF5258BB3694673EA52E25A1604.jpg)

2.到根目录下`sudo cat /etc/cron.d/sysstat` 输入密码，可以看到文件内容
![test3.jpg](/images/ST_Lab3JMeter/8B9DE06B31F9C7504F725AFC4A92B2F2.jpg)

3.打开该文件`sudo vim /etc/cron.d/sysstat` 把10改成1，让sa1 每隔一分钟执行一次
![test4.jpg](/images/ST_Lab3JMeter/0EEAE6631F21FD89B1828A7005A959E8.jpg)

4.在/var/log/sa下查看sysstat的统计报告，可以看到现在只有sa18，是每分钟的报告。
![test5.jpg](/images/ST_Lab3JMeter/1F4779E77A64E062838F47E62B0A652E.jpg)



### fio 评估磁盘读写性能极限
安装fio `yum install libaio-devel fio`

## ECShop 网上商店系统
[下载链接,仅供学习](http://pan.baidu.com/s/1dDAJvZN)
1.unzip下载的ECShop压缩包得到文件夹ECShop_V2.7.3_UTF8_release1106
里面有docs、upgrade、upload三个子目录
* docs: 安装的解释说明
* upgrade: 用于老版本的更新
* upload: 新用户需要复制  

`sudo cp -r upload /var/www/html`把upload复制到/var/www/html

2.修改ecshop文件夹的 owner，改成apache
`chown -R apache:apache /var/www/html`

3.关闭防火墙firewall：
`systemctl stop firewalld.service` 停止firewall
`systemctl disable firewalld.service` 禁止firewall开机启动

4.到ecshop目录下执行 `php -S 0.0.0.0:1024`

这时会发现：其实不用配置apache

访问 127.0.0.1:1024

出现页面
![test7.jpg](/images/ST_Lab3JMeter/CFA154E545FD77A5CF0F76C1685B2189.jpg)

按提示填写：
![test8.jpg](/images/ST_Lab3JMeter/6286AFF09834C05522C7BC7D374135B7.jpg)


5.安装完成后进入首页

![test9.jpg](/images/ST_Lab3JMeter/B976316729A3024A4DACB5C2B97243BE.jpg)
出现了一大堆错误提示：

`Strict Standards: Only variables should be passed by reference in /var/www/html/ecshop/includes/cls_template.php on line422`

解决办法：
打开cls_template.php文件中发现下面这段代码：
`$tag_sel = array_shift(explode(' ', $tag));`
我的PHP版本是5.4.16，PHP5.3以上默认只能传递具体的变量，而不能通过函数返回值传递，所以这段代码中的explode就得移出来重新赋值了。
```php
$tagArr = explode(' ', $tag);
$tag_sel = array_shift($tagArr);
```
这里可以vim该文件然后`:set nu`根据行号查找
这样之后顶部的报错没掉了，左侧和底部的报错还需要去[ecshop的后台](http://127.0.0.1:1024/admin/index.php)点击清除缓存才能去除。


`Strict Standards: Only variables should be passed by reference in /var/www/html/ecshop/includes/lib_main.php on line 1329`

解决办法：
lib_main.php中，把`$ext = end(explode('.', $tmp));`修改成 
```php
$tmp_arr=explode('.', $tmp);
$ext = end($tmp_arr);
```
![test10.jpg](/images/ST_Lab3JMeter/1D8369BE585BCACD827FB200F9601402.jpg)



## JMeter 压力测试工具
1.下载[JMeter](https://archive.apache.org/dist/jmeter/binaries/)

2.运行jmeter
`chmod +x jmeter`
`./jmeter `
出现窗口
![test11.jpg](/images/ST_Lab3JMeter/0BC38D83B237881462BD039EBDF244DD.jpg)

# 使用JMeter
## 录制 
1.工作台 -> 右键 添加  -> 非测试元件 -> HTTP代理服务器
需要录制的在"URL Patterns to Include"，不需要录制的在"URL Patterns to Exclude"，其中，点击“Add suggested Exclude”，显示录制时要屏蔽的内容`.*\.(bmp|css|js|gif|ico|jpe?g|png|swf|woff)`，即一些静态文件。

2.工作台 -> 右键 添加 -> 逻辑控制器 -> 交替控制器

3.交替控制器 ->  右键 添加 -> 配置元件  ->  HTTP请求默认值
它和sampler成对出现。只有先添加一个配置，才能去添加对应的sampler（请求）

4.交替控制器 -> 右键 添加 -> 逻辑控制器 -> 录制控制器

5.HTTP代理服务器 中的 目标控制器 用于存放录制的东西，一定要选择 “工作台 > 交替控制器”
填写Port代理服务器端口：8081。

6.监控数据 
HTTP代理服务器 -> 右键 添加 -> 监听器 ->  察看结果树

7.火狐浏览器配置代理服务器
首选项->高级->网络->设置，按下图填好，代理服务器端口号要和第5步里面一样
![test13.jpg](/images/ST_Lab3JMeter/6AF64E17CE1A9D2EC0657788C0ED5EFB.jpg)

这时会出现这个页面：
![test14.jpg](/images/ST_Lab3JMeter/415AFCD4F3A43E1D67EC24022A83BA18.jpg)

需要添加[证书](http://jmeter.apache.org/usermanual/component_reference.html#HTTP%28S%29_Test_Script_Recorder)来安装JMeterCA certificate for HTTPS recording。下面是在火狐安装certificate的过程：

首选项->高级项->证书->证书机构->添加
![test15.jpg](/images/ST_Lab3JMeter/E6916F5D44C4BD44A1F2DE60D036A963.jpg)

8.点击下方的“启动”，开始录制

9.打开ecshop （根目录首页、点击某个手机类型、点击某个物品详情），生成的动作在 交替控制器 里出现。

10.点击下方的“停止”，停止录制

## 回放
1.测试计划 -> 右键 添加 -> Thread user -> 线程组，命名为“线程组1”

2.线程组1 -> 右键 添加 -> 配置元件 ->  HTTP请求默认值
里面有ip、协议、path。有了它才能添加sampler，他们成对出现。 

3.把刚才生成的动作复制到 线程组1 下，保持顺序。

4.点击线程组，设置线程数为5，循环次数为10

5.线程组1 -> 右键 添加  -> 监听器 ->  聚合报告

6.线程组1 -> 右键 添加  -> 监听器 ->  察看结果树

7.停止浏览器里的代理服务器

8.保存test plan，执行start，上方绿色三角

9.执行top
![test18.jpg](/images/ST_Lab3JMeter/E24CAE1AA5262C3813A6851ADA1FBA47.jpg)

10.查看Aggregate Rreport 出现响应时间、数据量等。
![test19.jpg](/images/ST_Lab3JMeter/4D61701824E117ABE665F6DCBA910E76.jpg)

查看View Result Trees
![test17.jpg](/images/ST_Lab3JMeter/3AC090A4BA236E04BA461648F089767C.jpg)

# 测试脚本参数化
1.点击 /index.php 动作，其中ip和port需要参数化

2.点击 /category.php 动作，其中ip和port，类型id 需要参数化
打开左边的加号，出现HTTP Header Manager，其中referer的IP和port需要参数化

3.点击 /goods.php 动作，其中ip和port，货品id 需要参数化
打开类型动作左边的加号，出现HTTP Header Manager，referer需要参数化

4.线程组1 -> 右键 添加 -> 配置元件 -> 用户定义的变量

点击“添加” 设置两个用户变量和对应的值：
* IP  127.0.0.1
* PORT 1024

5.点击三个动作的 HTTP请求，将 “web服务器”框中 ServerNameOrIP改为`${IP}`，将PortNumber改为`${PORT}`

6.在类型和货品里，还需要将HTTP Header Manager中的Referer对应的值改为类似于 
`http://${IP}:${PORT}/index.php`
`http://${IP}:${PORT}/category.php?id=3`
`http://${IP}:${PORT}/goods.php?id=9`

7.bin目录下新建data目录，新建catgd.csv
其中有三种类型
* GSM手机
* 双模手机
* 手机配件

id分别是3，5，6
```csv
3,9
5,8
6,5
```
Encoding 设为 UTF-8 without BOM

8.线程组1 -> 右键 Add -> 配置元件 -> CSV Data Set Config
Filename:  `./data/catgd.csv`
File encoding: `utf-8`
Variable Names (comma delimited): `catid,gdid`
Delimiter (use 'u' for lab):` ,`
Allow quoted data?; `False`
Recycle on EOF ?: `True`
Stop thread on EOF ?: `False`
Sharing mode:`All threads`

9.点击三个动作的HTTP Request添加相应的id值

以/category.php为例：
![test21.jpg](/images/ST_Lab3JMeter/7AF5D096756C5EBDF54E02E767090AED.jpg)

在HTTP信息头管理器的referer中将相应的id值相应改为`${catid}`和`${gdid}`

以/category.php为例：
![test22.jpg](/images/ST_Lab3JMeter/6CD6891BFA057DFC1CA6D8FB1FBC72EA.jpg)

10.保存，点击上方两个并排的扫帚“ClearAll”按钮清理report，Start重新执行。察看树结果都为绿。

# 用PostProcessor获取响应中的关键数据

判断每个 sampler 返回的页面是否正确
用浏览器审查元素，比如：类型id为3的返回页面中，在“当前位置”之后检测到了“GSM手机”说明正确。

![test23.jpg](/images/ST_Lab3JMeter/77FB4338D10F0806E40122DB82D511A5.jpg)


1.点击 /category.php -> 右键 Add -> 后置处理器 -> CSS/JQuery Extractor
Reference Name:`ur_here`
CSS/JQuery expression:`#ur_here a:last-child`
Attribute:`href`
Match No.(0 for Random): `1`
Default Value:`na`

2.点击/category.php -> 右键 Add -> 断言 -> Bean Shell 断言，负责校验。
勾选“Reset bsh.Interpreter before each call”
Parameters (-> String Parameters and String[]bsh.args)  ` ${ur_here} ${catid}` 注意空格区分。

3.下方BeanShell脚本代码：

```java
java.util.regex.Pattern p=java.util.regex.Pattern.compile("id= (\\d+)");
//把要定位的节点的正则表达式预编译
java.util.regex.Matcher m=p.matcher(bsh.args[0]);
//匹配器检查有多少节点匹配定义的参数
boolean found= m.find();//查找匹配的节点
if(found){
  if(!m.group(1).equals(bsh.args[1])){//group为找到的结果集
  //m.group(1)中的1为第一个节点，是我们要的节点。bsh.arg[1]为Beanshell Assertion的parameters的第二个参数${brandid}，作为校验标准
    Failure=true;
    FailureMessage = m.group(1) + "<>" +bsh.args[1];//当Failure为true时，可输出错误信息 
  }
}
else{
  Failure = true;
  FailureMessage = "Not Found";
}
```
点击绿色三角运行。


4.保存，点击上方两个并排的扫帚“ClearAll”按钮清理report，Start重新执行。


# 5*10的测试结果

按照上述步骤得到以下结果
##Jmeter的Testplan展开截图为
![test26.jpg](/images/ST_Lab3JMeter/4C7EF0F73029004F664A52BA9CEBFF91.jpg)

##察看树结果为
![test24.jpg](/images/ST_Lab3JMeter/E0AA4D1EEDDEBE12942FC3BCB38742C2.jpg)

##聚合报告（Aggregate Report Result）截图为
![test25.jpg](/images/ST_Lab3JMeter/3FB0668999F4E97FE41219CA032136F8.jpg)

#  50*20的测试结果
将线程数和循环次数改为50和20，clear清除全部，重新运行
![test27.jpg](/images/ST_Lab3JMeter/AB76E10ECA033C66EF23A786981D1341.jpg)

## top
![test28.jpg](/images/ST_Lab3JMeter/1C6F867AB0441CDE4EF6CC60E9CCF119.jpg)

##察看树结果为
![test30.jpg](/images/ST_Lab3JMeter/A631B4B20FFB813EF4B93E9A29ED03E4.jpg)

##聚合报告（Aggregate Report Result）截图为
![test29.jpg](/images/ST_Lab3JMeter/5461C7B7ED0754FE0CE5C0B557B1447D.jpg)

## 任务数，繁重程度：
sysstat 的统计报告在/var/log/sa，sa指的是每分钟得到的报告，sar表示的是daily summery。今天的报告是sa21。
运行`sar -q -f sa21`得到：
![test31.jpg](/images/ST_Lab3JMeter/1E332468AFD073FE79EA9ADDA0FD3426.jpg)

19：56分左右时ldavg-1 > ldavg-5 > ldavg-15，说明这段时间CPU压力越来越大，这也正是测试执行时间，也就是刚才top截图中id为%0的时间。

## 查看CPU各项指标占用的百分比
运行`sar -p -f sa21` 得到：
![test32.jpg](/images/ST_Lab3JMeter/CFACEF9EA4A26E61CC65E991EFC86D05.jpg)
19：56分左右时%idle即空闲的百分比为0，%user特别高

## 内存实际占用百分比
运行`sar -r -f sa21`得到：
![test35.jpg](/images/ST_Lab3JMeter/45804C621AB5CC55F3C15744A3629285.jpg)
可以看出内存使用率一直都很高，但是19:55分56分的时候最高。

## 主存(内存)与块设备(硬盘)之间换页频繁程度
运行`sar -B -f sa21` 得到：
![test33.jpg](/images/ST_Lab3JMeter/FB92B59D9B89716CF98BD3AF5FC2C8CF.jpg)
可以看出，19:56分左右每秒从磁盘到内存的换页（换进）的字节数和每秒从内存到磁盘的换页（换出）的字节数都比以前大很多，意味着IO读写压力很大。当前，内存是瓶颈。

## 虚拟内存中,从块设备swap区中读入/读出的页数
运行`sar -w -f sa21`得到：
![test36.jpg](/images/ST_Lab3JMeter/6C78BC5210E9E95E9CBD48046418DD20.jpg)
19:55分56分的时候，虚拟内存中从块设备swap区中读入的页数，以及从块设备swap区中读出的页数都很大。

## IO的频繁程度
运行`sar -b -f sa21` 得到：
![test37.jpg](/images/ST_Lab3JMeter/E90CD22DF7CB138CF1BDF7BAE71817C0.jpg)
19:55分的时候，每秒钟物理设备的IO请求次数、每秒钟从物理设备读入读出的请求次数、以及每秒钟从物理设备读入读出的数据都量特别大。

## 扇区 
运行`sar -d -f sa21` 得到：
![test38.jpg](/images/ST_Lab3JMeter/F18C7E0156921940780985A728B08CAF.jpg)
可以看出，19:55分左右，%util即I/O请求占CPU的百分比很大，说明已经趋于饱和。另外，每秒钟物理设备的IO请求次数等数据也很高。


# 出现的问题
1.打开sysstat文件没有权限
![q1.jpg](/images/ST_Lab3JMeter/5918A96AD9542AA8E8592A9B412DB7F1.jpg)
问题原因：配置的时候“把自己设置为管理员”没有打勾。
解决方法：新建一个其他用户，这时“我的账号”就可以设置为“管理员”，新建的那个用户就可以删了，现在的账户就是管理员类型的账户了。重启后，再`sudo cat /etc/cron.d/sysstat` 输入密码就好了。

2.用 `yum list fio` 安装fio没有匹配的软件包
![q2.jpg](/images/ST_Lab3JMeter/E25EED4E78461A22A91B65D6EC7B4DAC.jpg)
解决办法： `yum install libaio-devel fio`

3.yum安装时显示 “/var/run/yum.pid 已被锁定，PID 为 xxxx 的另一个程序正在运行”
解决方法：`rm -f /var/run/yum.pid`

4.apache配置繁琐
配置apache服务器，需要修改conf文件，但是启动没有成功。

解决方法：
从php5.4开始，引入了一个内置web服务器，可以在测试环境迅速搭建web环境而无须复杂的配置。性能肯定不如nginx和apache服务器，生成环境还是要搭建服务器。
URI请求会被发送到PHP所在的工作目录（Working Directory）进行处理，或者可以使用-t参数来自定义不同的目录。
如果未指定执行哪个PHP文件，则默认执行该目录下的index.php 或index.html。
用`php -S 0.0.0.0:1024` 的命令来启动，然后在浏览器中访问`127.0.0.1:1024` 。



# 不该出现的问题

1.在录制阶段，JMeter可以记录访问百度等动作，却偏偏无法能记录Ecshop的任何动作。
问题原因：在火狐浏览器设置代理服务器时，手动配置代理中的最后一个“不使用代理”里没有删掉默认填写的“localhost”以及“127.0.0.1:1024”。
![error.jpg](/images/ST_Lab3JMeter/770BB61FCCBFE7CE1C7E0F3942535002.jpg)
把这些清空就好了。

2.察看树结果里category和good报错，聚合报告里显示所有category和good的出错率是100%。
详细信息如下图所示：
![error1.jpg](/images/ST_Lab3JMeter/1A4E0AF8D0582B6EB9C2951B5D831A01.jpg)
![error2.jpg](/images/ST_Lab3JMeter/90DA88A5B37D07BADA504DD4E0949B3B.jpg)
![error3.jpg](/images/ST_Lab3JMeter/229B24C589D6F892931FB0915FBACF34.jpg)
![error4.jpg](/images/ST_Lab3JMeter/D606D4F984B61ACB9FF839E020FB4AF7.jpg)
问题原因：经过分析，我发现JMeter获取的所有类型和货品的id号都显示为\<EOF\>，也就意味着程序获取的数据都为空。后来经过漫长的检查发现是在CSV Data Set Config中的 Filename:  `./data/catgd.csv` 写成了 `../data/catgd.csv` 因此JMeter并没有识别到该catgd.csv文件。

解决方案：在BeanShell断言里else部分补充为
```java
else {
	Failure = true;
	FailureMessage ="Not Found" ;
}
```
这样当没有识别到该文件时，就会出现断言错误结果：
![error5.jpg](/images/ST_Lab3JMeter/271894030CFFA2A833F56DAE478E5B87.jpg)
最后把 CSV Data Set Config中的 Filename:  `./data/catgd.csv` 修改正确。



