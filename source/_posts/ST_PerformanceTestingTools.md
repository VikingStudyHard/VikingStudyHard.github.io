---

title: 【软件测试】《高性能产品的必由之路—性能测试工具》课程笔记

date: 2018.04.10

categories: [软件测试, 本科课程]

tags: 

 - SoftwareTest

description: 开发人员如果具备了性能测试和性能优化的技能，在成长为架构师的路上会更加有竞争力。本门课程以Jmeter为中心给大家介绍了最最常用的性能测试的工具，通过对不同类型的系统进行性能测试，了解性能测试在真实项目中的意义，最后通过测试加压来观察和分析系统的瓶颈。

---

[高性能产品的必由之路—性能测试工具（慕课网网址）](http://www.imooc.com/learn/278)

# 1章 背景介绍
本章主要介绍了性能测试的产生背景。
## 1-1 性能测试背景介绍 (07:33)
横向扩展：如淘宝平时只开1000台服务器，双十一时会开5000台服务器，平时多出来的机器可用于“阿里云”服务器租赁。

数据库优化：当十万个用户使用数据库时就可能出现问题，需要优化数据库

让一台机器能够承载更多的用户请求
#2章 性能测试概要
本章主要讲解性能测试的相关概念和实际开发中的意义，并阐述了软件性能的一些关键性指标。
## 2-1 课程简介 (00:45)
## 2-2 关于性能的几个案例分析(05:02)
性能测试概要
案例：
* 反例：12306、2008奥运会订票
* 正例：淘宝双十一

实际情况超过了承载能力
## 2-3 性能的关键指标介绍 (11:03)
案例的共同特点：
* 同时有超多的用户使用网站服务
* 网站宕机

### 性能测试
通过技术的手段模拟大量用户**同时**访问被测应用，观察、记录和分析系统的**各项性能指标**的过程。
* 模拟大量并发用户
* 监控系统负载参数，分析系统瓶颈

性能测试的目标：评估系统的性能瓶颈，预测系统的最大用户承载能力。

### 性能指标
#### 平均响应时间(TTLB,Time to laster by)
平均每个请求从发送到接收响应的时间

![p1.png](/images/ST_PerformanceTestingTools/6B2873CF3A26828BA69635F86FC866EA.png)

##### 合理的平均响应时间：2/5/10原则
当用户能够在2秒以内得到响应时，会感觉系统的响应很快；
当用户在2-5秒之间得到响应时，会感觉系统的响应速度还可以；
当用户在5-10秒以内得到响应时，会感觉系统的响应速度很慢，但是还可以接受；
而当用户在超过10秒后仍然无法得到响应时，会感觉系统糟透了，或者认为系统已经失去响应，而选择离开这个Web站点，或者发起第二次请求。

所以：最好在在五秒以内
##### 平均响应时间业务影响

1秒的页面加载延迟相当于少了11%的PV(page view)，相当于降低了16%的顾客满意度。

页面响应时间从2秒增加到10秒会导致38%的页面浏览放弃率
#### 系统资源类的性能指标
* CPU（CPU的占用率）Apache进程的数量
* 内存（内存占用率、换页数等）用户量的增加
* I/O（读写请求数、读写量等）与数据库服务器有关。MySQL服务器，数据存在硬盘，最近、常用的数据缓存在内存，高并发用户访问时，真正瓶颈还是在I/O，I/O越少性能提高的效果越好
* 带宽（进站出站带宽占用率）和钱有关，要买新的带宽


## 2-4  为什么要进行性能测试 (01:59)
### 为什么要进行性能测试
能够有效评估系统的**性能指标**，用于系统的性能评估
能够识别系统的**性能瓶颈**，协助性能调优
能够指导**突发流量承载**方案的制定
能够用于系统运维**成本的预算**

## 2-5  性能测试分类 (03:57)
### 性能测试的一般分类
1.**负载测试**(Load Test)
为了验证系统设计符合**正常**业务负载情况下系统性能表现的测试。有明确的指标。
2.**压力测试**(Stress Test)
为了验证系统在**极端**负载情况下的性能表现的测试。（找到瓶颈和极限处）

负载测试由开发人员执行，压力测试由测试人员执行。

开发人员的性能测试更加关注在一定负载情况下系统资源占有率，从而找到内存泄露，链接泄露和系统的性能瓶颈。
#3章 性能测试相关工具
本章带领待解学习使用性能测试的相关工具。
## 3-1 使用Top获取进程的资源占用信息 (09:30)
### 准备工作
一台CentOS服务器（虚拟机）
一个可以连接服务器的SSH工具，推荐Putty （开源 无管制）
### 性能测试工具 top 
> 监控每一个进程的资源占用

打开putty

命令行中输入`top`
显示 系统状态信息、进程信息
Cpu中idle(空闲率)降到30%一下就说明CPU占用率特别高，就有问题了
* 帮助：`h`
* 调整顺序：`o`  显示出不同的字母对应的条目。大写字母往左移，小写字母往右移。*表示正在显示的条目。esc键退出。
* 排序(从大到小)：`F`或`O` 通过输入对应的字母来确定排序的条件。*星号表示正在使用的排序条件。输入任意键退出。
* 进程相关的条目有：
PID 进程的id
USER 打开进程的用户
PR priory 进程可被执行的优先级 值越小优先级越高，越早被执行。
NI nice 优先级的修正数值 PRI(new)=PRI(old)+nice
VIRT 虚拟内存的大小
RES 真实的物理内存的大小
SHR 共享内存的大小
%MEM 内存使用率
%CPU CPU使用率
TIME+ 真正运行的时长
COMMAND 具体运行的命令
* 参看具体说明 输入 `man`
* 查看所有的进程 `top -ab -n 1`
 a 按照内存的倒序排序
 b 把所有的列全部列出
 n 只执行一次，不会一直更新
* 过滤出httpd  `top -ab -n 1 | grep httpd`

## 3-2 性能统计工具sysstat安装配置 (06:00)
### 性能测试工具 sysstat
> 统计系统的各种资源的占用情况

*  安装sysstat `yum list sysstat` 相当于Ubuntu下的 `apt-get install sysstat` 没有“Installed Packages”时需要 `yum install sysstat.x86-64(安装包名字) -y`
y表示自动确认，不加的话会询问
*  sysstat工具集的配置文件位置 `cat /etc/cron.d/sysstat`
sysstat定时获取统计数据的时间间隔
其中

> /1 * * * * root /usr/lib64/sa/sa1 1 1

sa1 每隔一分钟执行一次，以root的身份来运行。
这里10要改成1。开发环境最好设置为1分钟，生产环境10分钟。
> 53 23 * * * root /usr/lib64/sa/sa2 -A

sa2 每天23:53生成日常总结报告。
* sysstat 的统计报告在/var/log/sa下
sa指的是每分钟得到的报告，sar表示的是daily summery

## 3-3 sysstat常用命令之CPU监控 (09:04)

###`sar -q -f sa25` 任务数，繁重程度
* runq-sz: Run queue length(number of tasks waiting for run time)等待执行的任务队列长度。越长阻塞越严重。
* plist-sz: number of task in the task list 队列中的任务总数
* ldavg-1,ldavg-5,ldavg-15: system load average for the last minute,1分钟、5分钟、15分钟内系统负载描述。
值是通过执行中任务和等待执行的任务的个数的平均值得到的。
ldavg-1 > CPU总数时，表示CPU压力大
ldavg-1 > ldavg-5 > ldavg-15 时说明当前CPU压力越来越大，需要注意。
ldavg-1 < ldavg-5 < ldavg-15 时说明当前CPU压力正在减小。

###`sar -p -f sa25` 查看CPU各项指标占用的百分比
* %user
* %nice: 改过优先级的进程CPU占用率
* %system
* %iowait: 等待磁盘读写的百分比。
* %steal: 管理程序(hypervisor)为另一个虚拟进程提供服务而等待虚拟CPU的百分比，等待CPU。值越大，CPU繁重程度越高。
* %idle: 空闲的百分比

## 3-4 sysstat常用命令之内存监控 (06:28)
### `sar -r -f sa25`  内存实际占用百分比
* kbmemfree: 空闲的内存
* kbmemused: 已经使用的内存
* %memused: 内存使用率
* kbbuffers: 文件的缓存 
* kbcached: 磁盘块的缓存
* kbcommit: 保证程序的的正常运行需要的内存
* %commit: 与memused的和若大于100%，则会导致内存频繁换页，使用虚拟内存

### `sar -B -f sa25` 主存(内存)与块设备(硬盘)之间换页频繁程度
* pgpgin/s: 每秒从磁盘到内存的换页（换进）的字节数KB。
* pgpgout/s: 每秒从内存到磁盘的换页（换出）的字节数KB。这两个数值如果很大，意味着IO读写压力很大，当前，内存是瓶颈。
* fault/s: 每秒系统产生的缺页数。内存里没有，要去磁盘里读取。等于major+minor
* majflt/s: 主缺页，必须发生换页
* pgfree/s 
* pgscank/s 
* pgsteal/s 
* %vmeff 

### `sar -w -f sa25` 指虚拟内存中,从块设备swap区中读入/读出的页数
* pswpin/s (page swap in)虚拟内存中,从块设备swap区中读入的页数
* pswpout/s (page swap out)虚拟内存中,从块设备swap区中读出的页数

## 3-5 sysstat常用命令之IO监控 (04:47)
### `sar -b -f sa25` IO的频繁程度
*  tps: 每秒钟物理设备的IO请求次数 tps=rtps+wtps 
*  rtps: 每秒钟从物理设备读入的请求次数
*  wtps: 每秒钟从物理设备写入的请求次数
*  bread/s: 每秒钟从物理设备读入的数据量，单位为 块/s
*  bwrtn/s: 每秒钟向物理设备写入的数据量，单位为 块/s

wtps比rtps多很多，需要降低写操作次数，如把文件打包一起写到磁盘里。
### `sar -d -f sa25` 扇区 
* tps 每秒钟物理设备的IO请求次数
* rd_sec/s: 每秒读扇区的次数
* wr_sec/s: 每秒写扇区的次数
* avgrq-sz: 平均每次请求操作的数据的扇区的大小。值越大，每次读写操作read的块特别大。
* avgqu-sz: 磁盘请求队列的平均长度。
* await: 从请求磁盘操作到系统完成处理，每次请求的平均消耗时间，单位是毫秒(1秒=1000毫秒)。
* svctm: 系统处理每次请求的平均时间，不包括请求队列中消耗的时间。
* %util: I/O请求占CPU的百分比，比率越大，说明越饱和。

## 3-6 sysstat常用命令之NetWork监控 (02:38)
### `sar -n DEV -f sa25` 网络数据交换的量
* DEV显示网络接口信息
* EDEV 显示关于网络错误的统计数据
* NFS统计活动的NFS客户站端的信息
* NFSD统计NFS服务器的信息
* OCK显示套接字信息
* ALL显示所有5个开关

它们可以单独或者一起使用。
* rxpck/s: 每秒钟接收的数据包
* txpck/s: 每秒钟发送的数据包
* rxkB/s: 每秒钟接收的字节数
* txkB/s: 每秒钟发送的字节数
* rxcmp/s: 每秒钟接收的压缩数据包
* txcmp/s: 每秒钟发送的压缩数据包
* rxmcst/s: 每秒钟接收的多播数据包 

### `sar -n NFS -f sa25` 网络错误
* IFACE: LAN 接口
* rxerr/s: 每秒钟接收的坏数据包
* txerr/s: 每秒钟发送的坏数据包
* coll/s: 每秒冲突数
* rxdrop/s: 因为缓冲充满，每秒钟丢弃的已接收数据包数
* txdrop/s: 因为缓冲充满，每秒钟丢弃的已发送数据包数
* txcarr/s: 发送数据包时，每秒载波错误数
* rxfram/s: 每秒接收数据包的帧对齐错误数
* rxfifo/s: 接收的数据包每秒FIFO过速的错误数
* txfifo/s: 发送的数据包每秒FIFO过速的错误数

## 3-7 评估磁盘读写性能极限 (04:27)
### 性能测试工具 fio
> 评估磁盘读写性能极限

* 安装fio `yum list fio` 没有“Installed Packages”时需要 `yum install fio.x86-64(安装包名字) -y`
* `fio -filename=/data/test(文件名) -direct=1-iodepth 1 -thread -rw=randrw -ioengine=psync -bs=16k -size 2G -numjobs=10 -runtime=30 -group_reporting -name=mytest13` 
* read里面有iops，指的是每秒钟读几次IO；write里的iops，指的是每秒钟写几次IO
* iops低的不适用做数据库服务器

## 3-8 JMeter性能测试工具简介 (01:18)
### 性能测试工具 JMeter 
> Apache 开源的
用于生成大量并发的请求

# 4章 被测系统准备
本章主要带领大家一起安装学习待测系统，为后续的性能测试做准备。
## 4-1 被测系统ECShop介绍 (01:59)
### ECShop 简介
开源的网上商店系统，商用付费
## 4-2 被测系统ECShop安装 (18:32)
### ECShop 安装
[下载链接,仅供学习](http://pan.baidu.com/s/1dDAJvZN)
官网上，下载非商业授权版

### WinSCP 上传本地文件到服务器
（FTP也可以，WinSCP用的是SFTP也就是SSH）
1.登录服务器
会话 -> 新建会话 -> 选择服务器 -> 编辑 -> 填入主机名，端口号，用户名-> 登录

2.上传本地文件到服务器
拖动文件到某个目录，传输设置选择“二进制”，点击“复制”

3.用putty登录服务器
unzip下载的ECShop压缩包得到文件夹ECShop_V2.7.3_UTF8_release1106
里面有docs、upgrade、upload三个子目录
* docs: 安装的解释说明
* upgrade: 用于老版本的更新
* upload: 新用户需要复制  `cp -r upload /ecshop`

4.修改ecshop文件夹的 owner，改成apache
`chown -R apache:apache ecshop`

5.配置apache服务器
在 /etc/httpd/conf.d 目录下创建 ecshop.conf 文件
```
Listen 83                //83是监听的端口号，所有webapp默认端口都是80
<VirtualHost *:83>                                     //创建虚拟主机
  ServerAdmin oldleader@163.com
  DocumentRoot /ecshop                                       //根目录
  ErrorLog /ecshop/log/error_log                         //出错的日志
  CustomLog /ecshop/log/access_log common //正常访问的日志 common是级别
    <Directory "/data/html/ecshop">                      //目录权限
      Options Indexes FollowSymLinks
      AllowOverride All
      Order allow,deny
      Allow from all
    </Directory>
</VirtualHost>
```

6.在ecshop文件夹中新建log文件夹`mkdir log`并修改owner `chown -R apache:apache log`

7.重启服务器apache，访问83端口
`service httpd restart`
访问http://.........:83

8.按照提示修改php里timezone等东西
在开头注释之前加上`date_default_timezone_set('PRC);`保存后刷新页面。

9.配置数据库账号、管理员账号、禁用验证码，安装测试数据

## 4-3 被测系统功能简介 (05:35)
前往ecshop首页（就是那个83端口）
开发人员找到某个页面施压，测试人员都整体网站施压
注册会员，登录。

#5章 使用Jmeter进行性能测试
本章带领大家从录制脚本开始使用Jmeter进行系统的性能测试。
## 5-1 JMeter工作原理 (04:02)
JMeter

![p2.png](/images/ST_PerformanceTestingTools/0F542BC62FAF87FB2A32979E4025B8CB.png)
thread 线程 **一个thread代表一个用户**
sampler 代表每个请求，采样

## 5-2  JMeter下载安装及其目录结构说明 (05:58)
[JMeter官网](jmeter.apache.org)
点击Download Releases，下载2.11 Binaries
## 5-3 JMeter基本功能介绍 (16:37)

目录bin中，打开jmeter.bat
* Test plan：测试计划，以什么样的方式去执行测试
* WorkBench：工作区，可以预编辑

Test plan 右键 Add -> Threads(Uesrs) ->  Thread Group
Thread Group用户组，代表具有相同行为属性的用户

目录bin中，打开jmeter.properties可以设置语言环境 22行改成`language=en`保存后重新打开jmeter.bat。

可以设置“请求后遇到错误的处理方式”：
* continue:继续执行
* Start Next Thread Loop 重新执行
* Stop Thread 停掉线程
* Stop Test 停掉测试(等已经开始的线程全部结束之后再停止）
* Stop Test Now 立即停掉测试

参数：
* 模拟用户的人数
* 在多长时间内把全部任务都启动起来。一般设为1，不用阶梯递增。
* 执行次数，选择forever，循环发送请求，除非有登录
* schedule 执行测试计划，勾选，并填空


在新建的Thread Group上 右键 右键 Add -> Sampler 请求->  一般选择HTTP Request
Add -> Logic Controller ->  随机执行等逻辑控制器
Add -> Config Element 配置信息
Add -> Pre Processors 预处理
Add -> Timer 定时器 ->  ConstantTimer 等 设置执行时间
Add -> Post Processors 后续处理
Add -> Assertions 断言
Add -> Listener 监听器 ->  Simple Data Writer等 执行结果、性能

Run里面
* Start 开始
* Remote Start 分步测试某一个，用于调试
* Remote Start All 分步测试所有
* Clear 初始化
* ClearAll 全部初始化


## 5-4 使用Jmeter对性能测试脚本进行录制和回放 (22:43)

自动化的一般方法
### 录制


1.WorkBench -> 右键 Add  -> Non-Test Elements ->HTTP(S) Test Script Recorder
是一个代理服务器
需要录制的在"URL Patterns to Include"，不需要录制的在"URL Patterns to Exclude"，其中，点击“Add suggested Exclude”，显示录制时要屏蔽的内容`.*\.(bmp|css|js|gif|ico|jpe?g|png|swf|woff)`，即一些静态文件

2.WorkBench -> 右键 Add  -> Logic Controller -> Transaction Controller

3.Transaction Controller -> 右键 Add -> Config Element  ->  HTTP Request Defaults，他和sampler成对出现。只有先添加一个配置，才能去添加对应的sampler（请求）

4.Transaction Controller -> 右键 Add -> Logic Controller  -> Recording Controller

5.HTTP(S) Test Script Recorder 中的 targetController 用于存放录制的东西，选择WorkBench > Transaction Controller

Port 代理服务器端口
6.Start

7.代理服务器 IE浏览器 -> 工具 -> Internet 选项 -> 连接 -> 局域网设置 -> 代理服务器 填好 localhost 端口

8.监控数据 
HTTP(S) Test Script Recorder -> 右键 Add -> Listener ->  View Result Trees

9.打开ecshop（根目录首页、点击某个手机类型、点击某个物品详情），生成的动作在Transaction Controller里出现
### 回放
 
1.用户组1 -> 右键 Add -> Config Element  ->  HTTP Request Defaults
里面有ip、协议、path。有了它才能添加sampler，他们成对出现。 

2.把动作复制到用户组1下，保持顺序。

3.点击用户组1，数据都设为1

4.用户组1 -> 右键 Add  -> Listener ->  Aggregate Rreport

5.用户组1 -> 右键 Add  -> Listener ->  View Result Trees
6.停止录制，停止浏览器里的代理服务器
7.保存test plan，执行start，上方绿色三角
8.Aggregate Rreport 出现响应时间、数据量等。
9.View Result Trees

## 5-5 测试脚本参数化 (15:23)
1.点击根目录动作，ip和port需要参数化

2.点击类型动作，ip和port，类型id需要参数化
打开左边的加号，出现HTTP Header Manager，其中referer的IP和port需要参数化

3.点击货品动作，ip和port，货品id 需要参数化
打开类型动作左边的加号，出现HTTP Header Manager，referer需要参数化

4.用户组1 -> 右键 Add -> Config Element -> User Defined Variables

Add 设置两个用户变量和对应的值：
* IP 
* PORT

5.点击三个动作的HTTP Request，将ServerNameOrIP改为`${IP}`，将PortNumber改为`${PORT}`
6.在类型和货品里，还需要将HTTP Header Manager中的Referer对应的值改为类似于 `http://${IP}:${PORT}`

7.bin目录下新建data目录，新建catgd.csv
其中有三种类型
* GSM手机
* 双模手机
* 手机配件

id分别是3，5，6
```csv
3,9
5,8
6,3
```
Encoding 设为 UTF-8 without BOM

8.用户组1 -> 右键 Add -> Config Element -> CSV Data Set Config
Filename:  `./data/catgd.csv`
File encoding: `utf-8`
Variable Names (comma delimited): `catid,gdid`
Delimiter (use 'u' for lab):` ,`
Allow quoted data?; `False`
Recycle on EOF ?: `True`
Stop thread on EOF ?: `False`
Sharing mode:`All threads`

9.点击三个动作的HTTP Request、referer，将id的值相应改为`${catid}`和`${gdid}`

10.保存，点击上方两个并排的扫帚“ClearAll”按钮清理report，Start重新执行

## 5-6 用PostProcessor获取响应中的关键数据 (19:21）
判断每个 sampler 返回的页面是否正确
用浏览器审查元素，比如GSM手机对应的类型id的返回页面中检测到了“GSM手机”说明正确。
### CSS/JQuery Extractor
1.点击category.php -> 右键 Add -> Post Processors -> CSS/JQuery Extractor

Reference Name:`ur_here`
CSS/JQuery expression:`#ur_here a:last-child`
Attribute:`href`
Match No.(0 for Random): `1`
Default Value:`na`

2.点击category.php -> 右键 Add -> Assertion -> Bean Shell Assertion  
负责校验
勾选Reset bsh.Interpreter before each call
Parameters (-> String Parameters and String[]bsh.args)  ` ${ur_here} ${catid}`空格区分

3.下方脚本
```
print(bsh.args[0]);//打印${ur_here}，1的话打印${catid}
``` 
start执行

建议用这个

### Regular Expression Extractor
点击category.php -> 右键 Add -> Post Processors -> Regular Expression Extractor

Reference Name:`ur_here`
Reguar Expression:`id="(ur_here)"`
Template:`$1$`第一个自查询的结果
Match No.(0 for Random): `1`
Default Value:`na`

太难了不建议用

### XPath Extractor
1.点击category.php -> 右键 Add -> Post Processors -> XPath Extractor
勾选 use tidy
Reference Name:`ur_here`
XPath query:`//div[@id="ur_here"]`
Default Value:`na`

必须在html写得很规范时才能用

## 5-7 用BeanShellAssertion判断响应是否正确 (13:13)

BeanShell轻量级java脚本语言
BeanShell接收用空格区分的参数，用Java处理，要写清楚类
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
else Failure = true;
```

start运行

## 5-8 用BSF Assertion判断响应是否正确 (06:48)

BSFAssertion
language:`javascript`
parameters: `${catid} ${gdid}`

```java
OUT.println (args[0]);
var re = new RegExp("id= (\\d+) ","ig");
var arr= re.exec(args[0]);
if (!RegExp.$1 || RegExp.$1 == args[1]){
  AssertionResult.setFailure(true);
  AssertionResult.setFailureMessage(RegExp.$1+"<>" + args[1]);
}
```

## 5-9 JMeter中的其他断言方法 (02:08)
略
## 5-10 JMeter中模拟用户的等待行为 (06:13)
模拟用户点击之间的停留时间
用户组1 -> 右键 Add -> Timer -> Bean Shell Timer -> Constant timer
用于在Sampler A和Sampler B之间的等待时间
Poisson和Gussian Timer比较符合用户的实际情况

顺序如图：
![test12.png](/images/ST_PerformanceTestingTools/CB30A7A7DBE174E183C8F90FEAC8E776.png)

## 5-11 执行性能测试并在执行过程中监控性能 (11:46)
调整用户组数据：
线程数量设为100，第二行填1，勾选forever，保存
clear all
top
启动Start
看top的实时性能监控
停止Stop


## 5-12 本章小结 (01:20)
略





