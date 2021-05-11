---

title: 【服务器】在服务器上跑python代码
date: 2019.11.15
categories: 
 - [环境配置]

tags: 
 - Mac
 - RemoteServer


description: 连接服务器实录
---


# 连接服务器
> https://www.jianshu.com/p/2ee2434d7c28
> https://linux.cn/article-3858-1.html#3_487

SSH 默认连接到目标主机上
```
ssh user@hostname
```
添加参数 -p 指定端口号为 10022
```
ssh -p 10022 user@hostname
```

# 转移数据
先连接服务器创建文件夹`Projects`，修改权限`chmod 777 Projects`，登出。

scp 命令
-P 端口
-p 拷贝文件的时候保留源文件建立的时间
-q 执行文件拷贝时，不显示任何提示消息
-r 拷贝整个目录
-v 拷贝文件时，显示提示信息

1.获取远程服务器上的文件夹
```
scp -P 2222 -r user@hostname:远程保存路径 本地文件路径
```
端口大写P为参数，2222 表示更改SSH端口后的端口，如果没有更改SSH端口可以不用添加该参数。 

2.拷贝本地文件夹到远程服务器
```
scp -r 本地文件路径 user@hostname:远程保存路径
```

# 安装 anaconda 并配置默认环境
1.下载安装脚本
```
wget https://repo.anaconda.com/archive/Anaconda3-5.2.0-Linux-x86_64.sh
```
2.运行安装向导
```
bash Anaconda3-5.2.0-Linux-x86_64.sh
```

安装过程中会询问：是否将路径追加输入bashrc中，回复yes,`.bashrc`文件中会多出两行：

```
# added by Anaconda3 installer
export PATH="/home/user/anaconda3/bin:$PATH"
```

启用`.bashrc`

```
source ~/.bashrc
```
3.确认是否安装成功
```
conda --version
```

{% img https://vikingstudyhard.github.io/images/RemoteServer_FirstTry/F161FC2B0CBB63944B4C0FE801526CCF.jpg 530}

python成功启用anaconda的版本。

4.在base默认环境中安装包
```
conda install <package_name>
```


`PackagesNotFoundError: The following packages are not available from current channels:` 解决方法：
1.查找包
```
anaconda search -t conda 要安装的包
```
2.查看包的详细信息
```
anaconda show 列表中对应的Name
```
3.按照最后一行提示安装
```
conda install --channel https://conda.anaconda.org/列表中对应的Name 要安装的包
```
4.如果还是没有对应的安装包用：
```
pip install 要安装的包
```
> https://blog.csdn.net/miao0967020148/article/details/85230430
> https://blog.csdn.net/EWBA_GIS_RS_ER/article/details/84671406

5.获取当前环境中已安装的包信息
```
conda list
```


`ModuleNotFoundError: No module named 'pandas.core.internals.managers'; 'pandas.core.internals' is not a package`解决方法：

```
python3 -m pip install --upgrade pandas
```

# CPU or CUDA
```
AssertionError:
Found no NVIDIA driver on your system. Please check that you
have an NVIDIA GPU and installed a driver from
http://www.nvidia.com/Download/index.aspx
```
1.查看Linux系统版本信息
```
cat   /proc/version
```
得到
```
Linux version 4.15.0-45-generic (buildd@lgw01-amd64-031) (gcc version 7.3.0 (Ubuntu 7.3.0-16ubuntu3)) #48-Ubuntu SMP Tue Jan 29 16:28:13 UTC 2019
```
显示机器上的显卡信息
```
lspci | grep -i vga
lspci | grep -i nvidia
```

{% img https://vikingstudyhard.github.io/images/RemoteServer_FirstTry/7F3B0F9FAB5DFFF69210795546399746.jpg 632}

CUDA是NVIDIA推出的用于自家GPU的并行计算框架，也就是说CUDA只能在NVIDIA的GPU上运行，而且只有当要解决的计算问题是可以大量并行计算的时候才能发挥CUDA的作用。

而且这服务器并没有NVIDIA显卡。要用CPU训练。 `USE_GPU`的值改为`False`


# 运行
保存输出结果
```
命令 > xxx.txt
```

# git命令行入门教程

Git 全局设置:
```
git config --global user.name "Viking"
git config --global user.email "905652564@qq.com"
```

创建 git 仓库:
```
mkdir programName
cd programName
git init
touch README.md
git add README.md
git commit -m "first commit"
git remote add origin git@gitee.com:xxx/git_repo.git
git push -u origin master
```
已有仓库?
```
cd existing_git_repo
git remote add origin git@gitee.com:xxx/existing_git_repo.git
git push -u origin master
```
