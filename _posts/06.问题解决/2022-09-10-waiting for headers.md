# 问题描述

Unbutu 在安装软件的过程中，出现 waiting for headers，然后卡住一直没反应



# 解决办法

```
rm -rf /var/lib/apt/lists/* 
rm -rf /var/lib/apt/lists/partial/*
cd /var/cache/apt/archives  
sudo rm -rf partial
```



以上命令都是清理缓存文件。

如果清除缓存文件之后还是遇到waiting for headers，那就更换国内源

```bash
#备份现有源
sudo cp /etc/apt/sources.list /etc/apt/sources.list.bak 

#查看当前版本
cat /etc/issue  

#修改文件，替换下面软件源，删除文件所有内容，替换对应版本的源，见Ubuntu国内源
sudo vim /etc/apt/sources.list 

#更新列表
sudo apt-get update 
```



# Ubuntu国内源

## Ubuntu 14.04 (trusty)

### 阿里源

```bash
cat << "EOF" > /etc/apt/sources.list 
deb https://mirrors.aliyun.com/ubuntu/ trusty main restricted universe multiverse
deb-src https://mirrors.aliyun.com/ubuntu/ trusty main restricted universe multiverse
deb https://mirrors.aliyun.com/ubuntu/ trusty-security main restricted universe multiverse
deb-src https://mirrors.aliyun.com/ubuntu/ trusty-security main restricted universe multiverse
deb https://mirrors.aliyun.com/ubuntu/ trusty-updates main restricted universe multiverse
deb-src https://mirrors.aliyun.com/ubuntu/ trusty-updates main restricted universe multiverse
deb https://mirrors.aliyun.com/ubuntu/ trusty-backports main restricted universe multiverse
deb-src https://mirrors.aliyun.com/ubuntu/ trusty-backports main restricted universe multiverse
## Not recommended
# deb https://mirrors.aliyun.com/ubuntu/ trusty-proposed main restricted universe multiverse
# deb-src https://mirrors.aliyun.com/ubuntu/ trusty-proposed main restricted universe multiverse
EOF
```

### 网易源

```bash
cat << "EOF" > /etc/apt/sources.list 
deb http://mirrors.163.com/ubuntu/ trusty main restricted universe multiverse
deb http://mirrors.163.com/ubuntu/ trusty-security main restricted universe multiverse
deb http://mirrors.163.com/ubuntu/ trusty-updates main restricted universe multiverse
deb http://mirrors.163.com/ubuntu/ trusty-proposed main restricted universe multiverse
deb http://mirrors.163.com/ubuntu/ trusty-backports main restricted universe multiverse
deb-src http://mirrors.163.com/ubuntu/ trusty main restricted universe multiverse
deb-src http://mirrors.163.com/ubuntu/ trusty-security main restricted universe multiverse
deb-src http://mirrors.163.com/ubuntu/ trusty-updates main restricted universe multiverse
deb-src http://mirrors.163.com/ubuntu/ trusty-proposed main restricted universe multiverse
deb-src http://mirrors.163.com/ubuntu/ trusty-backports main restricted universe multiverse
EOF
```



## Ubuntu 16.04 (xenial)

### 阿里源

```bash
cat << "EOF" > /etc/apt/sources.list 
deb https://mirrors.aliyun.com/ubuntu/ xenial main
deb-src https://mirrors.aliyun.com/ubuntu/ xenial main
deb https://mirrors.aliyun.com/ubuntu/ xenial-updates main
deb-src https://mirrors.aliyun.com/ubuntu/ xenial-updates main
deb https://mirrors.aliyun.com/ubuntu/ xenial universe
deb-src https://mirrors.aliyun.com/ubuntu/ xenial universe
deb https://mirrors.aliyun.com/ubuntu/ xenial-updates universe
deb-src https://mirrors.aliyun.com/ubuntu/ xenial-updates universe
deb https://mirrors.aliyun.com/ubuntu/ xenial-security main
deb-src https://mirrors.aliyun.com/ubuntu/ xenial-security main
deb https://mirrors.aliyun.com/ubuntu/ xenial-security universe
deb-src https://mirrors.aliyun.com/ubuntu/ xenial-security universe
EOF
```

### 网易源

```bash
cat << "EOF" > /etc/apt/sources.list 
deb http://mirrors.163.com/ubuntu/ xenial main restricted universe multiverse
deb http://mirrors.163.com/ubuntu/ xenial-security main restricted universe multiverse
deb http://mirrors.163.com/ubuntu/ xenial-updates main restricted universe multiverse
deb http://mirrors.163.com/ubuntu/ xenial-proposed main restricted universe multiverse
deb http://mirrors.163.com/ubuntu/ xenial-backports main restricted universe multiverse
deb-src http://mirrors.163.com/ubuntu/ xenial main restricted universe multiverse
deb-src http://mirrors.163.com/ubuntu/ xenial-security main restricted universe multiverse
deb-src http://mirrors.163.com/ubuntu/ xenial-updates main restricted universe multiverse
deb-src http://mirrors.163.com/ubuntu/ xenial-proposed main restricted universe multiverse
deb-src http://mirrors.163.com/ubuntu/ xenial-backports main restricted universe multiverse
EOF
```



## Ubuntu 18.04(bionic)

### 阿里源

```bash
cat << "EOF" > /etc/apt/sources.list 
deb https://mirrors.aliyun.com/ubuntu/ bionic main restricted universe multiverse
deb-src https://mirrors.aliyun.com/ubuntu/ bionic main restricted universe multiverse
deb https://mirrors.aliyun.com/ubuntu/ bionic-security main restricted universe multiverse
deb-src https://mirrors.aliyun.com/ubuntu/ bionic-security main restricted universe multiverse
deb https://mirrors.aliyun.com/ubuntu/ bionic-updates main restricted universe multiverse
deb-src https://mirrors.aliyun.com/ubuntu/ bionic-updates main restricted universe multiverse
# deb https://mirrors.aliyun.com/ubuntu/ bionic-proposed main restricted universe multiverse
# deb-src https://mirrors.aliyun.com/ubuntu/ bionic-proposed main restricted universe multiverse
deb https://mirrors.aliyun.com/ubuntu/ bionic-backports main restricted universe multiverse
deb-src https://mirrors.aliyun.com/ubuntu/ bionic-backports main restricted universe multiverse
EOF
```



### 网易源

```bash
cat << "EOF" > /etc/apt/sources.list 
deb http://mirrors.163.com/ubuntu/ bionic main restricted universe multiverse
deb http://mirrors.163.com/ubuntu/ bionic-security main restricted universe multiverse
deb http://mirrors.163.com/ubuntu/ bionic-updates main restricted universe multiverse
deb http://mirrors.163.com/ubuntu/ bionic-proposed main restricted universe multiverse
deb http://mirrors.163.com/ubuntu/ bionic-backports main restricted universe multiverse
deb-src http://mirrors.163.com/ubuntu/ bionic main restricted universe multiverse
deb-src http://mirrors.163.com/ubuntu/ bionic-security main restricted universe multiverse
deb-src http://mirrors.163.com/ubuntu/ bionic-updates main restricted universe multiverse
deb-src http://mirrors.163.com/ubuntu/ bionic-proposed main restricted universe multiverse
deb-src http://mirrors.163.com/ubuntu/ bionic-backports main restricted universe multiverse
EOF
```



## Ubuntu 20.04(focal) 

### 阿里源

```bash
cat << "EOF" > /etc/apt/sources.list 
deb https://mirrors.aliyun.com/ubuntu/ focal main restricted universe multiverse
deb-src https://mirrors.aliyun.com/ubuntu/ focal main restricted universe multiverse
deb https://mirrors.aliyun.com/ubuntu/ focal-security main restricted universe multiverse
deb-src https://mirrors.aliyun.com/ubuntu/ focal-security main restricted universe multiverse
deb https://mirrors.aliyun.com/ubuntu/ focal-updates main restricted universe multiverse
deb-src https://mirrors.aliyun.com/ubuntu/ focal-updates main restricted universe multiverse
# deb https://mirrors.aliyun.com/ubuntu/ focal-proposed main restricted universe multiverse
# deb-src https://mirrors.aliyun.com/ubuntu/ focal-proposed main restricted universe multiverse
deb https://mirrors.aliyun.com/ubuntu/ focal-backports main restricted universe multiverse
deb-src https://mirrors.aliyun.com/ubuntu/ focal-backports main restricted universe multiverse
EOF
```



### 网易源

```bash
cat << "EOF" > /etc/apt/sources.list 
deb http://mirrors.163.com/ubuntu/ focal main restricted universe multiverse
deb http://mirrors.163.com/ubuntu/ focal-security main restricted universe multiverse
deb http://mirrors.163.com/ubuntu/ focal-updates main restricted universe multiverse
deb http://mirrors.163.com/ubuntu/ focal-proposed main restricted universe multiverse
deb http://mirrors.163.com/ubuntu/ focal-backports main restricted universe multiverse
deb-src http://mirrors.163.com/ubuntu/ focal main restricted universe multiverse
deb-src http://mirrors.163.com/ubuntu/ focal-security main restricted universe multiverse
deb-src http://mirrors.163.com/ubuntu/ focal-updates main restricted universe multiverse
deb-src http://mirrors.163.com/ubuntu/ focal-proposed main restricted universe multiverse
deb-src http://mirrors.163.com/ubuntu/ focal-backports main restricted universe multiverse
EOF
```



# sources.list格式

[参考文档](https://ywnz.com/linuxjc/1827.html)

## deb和deb-src的区别

```
deb后面跟着的是二进制包仓库地址
deb-src后面跟着的是源代码包仓库。通过deb-src可以下载源码版本的软件包

除非我们要参与软件的打包中，或者我们想获取软件的源码，所以我们只关注deb相关的那些行即可
```



## deb格式介绍

```bash
deb https://mirrors.aliyun.com/ubuntu/ trusty main restricted universe multiverse
deb https://mirrors.aliyun.com/ubuntu/ trusty-security main restricted universe multiverse
deb https://mirrors.aliyun.com/ubuntu/ trusty-updates main restricted universe multiverse
deb https://mirrors.aliyun.com/ubuntu/ trusty-backports main restricted universe multiverse
deb https://mirrors.aliyun.com/ubuntu/ trusty-proposed main restricted universe multiverse


#deb后面跟着的其实有三大部分：deb repos_path section1 section2
```



### repos_path

源的路径，deb可支持http，ftp,以及本地文件cdrom等路径。

我们访问https://mirrors.aliyun.com/ubuntu/可以看到如下图所示

![image-20220910214436243](https://raw.githubusercontent.com/wg1217/picbed/main/202209132351212.png)

比较重要的就是**dists**目录和**pool**目录。

#### **dists**

```
存放当前源的所有软件包的索引，按照section1命名为不同的文件夹，索引文件最后指向pool
```



#### **pool**

```
存放对应的软件包
```



### section1

#### 版本代号

**进入dists目录，看到如下图所示内容**

![image-20220910205116017](https://raw.githubusercontent.com/wg1217/picbed/main/202209132351213.png)



可以看到很多文件夹，这些文件夹都是以下格式命名的：

```
版本代号
版本代号-backports
版本代号-proposed
版本代号-security
版本代号-updates
```

其中版本代号就是ubuntu不同发行版本，可在[官网](https://wiki.ubuntu.com/Releases)查询到不同版本对应的版本代号

![image-20220910210007225](https://raw.githubusercontent.com/wg1217/picbed/main/202209132351214.png)

可以看到Ubuntu 20.04版本对应的代号是Focal



#### 不同命名的文件夹区别

[参考文档](https://forum.ubuntu.org.cn/viewtopic.php?t=253103)

##### 版本代号

由于ubuntu是每6个月发行一个新版，当发行后，所有软件包的版本在这六个月内将保持不变，即使是有新版都不更新。除开重要的安全补丁外，所有新功能和非安全性补丁将不会提供给用户更新。



##### 版本代号-security

```
Important Security Updates
仅修复漏洞，并且尽可能少的改变软件包的行为，低风险
```



##### 版本代号-backports

```
Unsupported Updates
backports的团队认为最好的更新策略是 security 策略加上新版本的软件（包括候选版本的），但不会由Ubuntu security team审查和更新。
```



##### 版本代号-updates

```
Recommended Updates
修复严重但不影响系统安全运行的漏洞，这类补丁在经过QA人员记录和验证后才提供，和security那类一样低风险。
```



##### 版本代号-proposed

```
Pre-released Updates
update类的测试部分，仅建议提供测试和反馈的人进行安装
```



### section2

打开bionic文件夹，可以看到如下内容：

![image-20220910221254085](https://raw.githubusercontent.com/wg1217/picbed/main/202209132351215.png)

有**main,multiverse,restricted,universe**等文件夹，这就是section2的内容



这些文件夹里面包含了不同软件包索引。这几个文件夹的包的区别在于

#### main

```
完全的自由软件
```



#### restricted

```
不完全的自由软件
```



#### universe    	

```
官方不提供支持与补丁，全靠社区支持。
```



#### muitiverse		

```
非自由软件，完全不提供支持和补丁。
```



 

我们打开main目录下的binary-i386子目录下的Packages.gz文件，下载解压打开可以看到如下内容:

![image-20220910221550446](https://raw.githubusercontent.com/wg1217/picbed/main/202209132351216.png)

![image-20220910215740929](https://raw.githubusercontent.com/wg1217/picbed/main/202209132351217.png)



这个文件其实是一个**索引**，里面记录了各种包的  包名(Package)、运行平台（Architecture）、版本号（Version）、依赖关系（Depends）、deb包地址（Filename）等

注意到 Filename指向的是服务器pool目录下的某个deb。

打开pool文件夹可以看到如下内容，正好与section2对应

![image-20220910214813119](https://raw.githubusercontent.com/wg1217/picbed/main/202209132351218.png)

apt-get install 某个软件时，其实就是基于这些Packages来计算依赖关系，根据其中的Filename地址来下载所需的deb，然后再执行dpkg -i xxx.deb来完成软件包的安装。

 



## **编写适合sources.list**

我们只要填写好respos_path,section1和section2即可。

 

### 获取ubuntu的版本代号

|   版本    |                    版本代号                    |
| :-------: | :--------------------------------------------: |
| **20.04** |  [Focal](https://wiki.ubuntu.com/FocalFossa)   |
| **18.04** | [Bionic](https://wiki.ubuntu.com/BionicBeaver) |
| **16.04** | [Xenial](https://wiki.ubuntu.com/XenialXerus)  |
| **14.04** |  [Trusty](https://wiki.ubuntu.com/TrustyTahr)  |



### 确定repos_path

国内常用的几大源有：

阿里云  http://mirrors.aliyun.com/ubuntu/

网易     http://mirrors.163.com/ubuntu/

搜狐     http://mirrors.sohu.com/ubuntu/

中科大 https://mirrors.ustc.edu.cn/ubuntu/ 

清华 https://mirrors.tuna.tsinghua.edu.cn/ubuntu/



 

### **确定section1**

打开 http://mirrors.163.com/ubuntu/，如果看下如下目录结构（尤其是dists目录和pool目录），说明该源可用

![image-20220910222822110](https://raw.githubusercontent.com/wg1217/picbed/main/202209132351219.png)

打开dists目录，检查该源是否包含可供ubuntu20.04使用的软件包。

![image-20220910222918495](https://raw.githubusercontent.com/wg1217/picbed/main/202209132351220.png)

可以看到以focal-* 为名的文件夹，这说明网易源现在支持20.04。

现在 我们可以将focal-*填入section1

**注意： 一条deb源只能填入一个section1，如果想添加多个源就需要另一行重新书写新的deb源。**

于是我们目前确认以下sources.list的内容：

```
deb http://mirrors.163.com/ubuntu focal section2
deb http://mirrors.163.com/ubuntu focal-updates section2
deb http://mirrors.163.com/ubuntu focal-security section2
deb http://mirrors.163.com/ubuntu focal-proposed section2
deb http://mirrors.163.com/ubuntu focal-backports section2
```



### 确定section2

依次打开bionic,bionic-updates,bionic-security,bionic-proposed目录，看包含哪几大类属性的软件。

例如focal目录

![image-20220910223254115](https://raw.githubusercontent.com/wg1217/picbed/main/202209132351221.png)

将main multiverse restricted universe填入section2即可。

最终sources.list的内容为：

```
deb http://mirrors.163.com/ubuntu focal main restricted universe multiverse
deb http://mirrors.163.com/ubuntu focal-updates main restricted universe multiverse
deb http://mirrors.163.com/ubuntu focal-security main restricted universe multiverse
deb http://mirrors.163.com/ubuntu focal-proposed main restricted universe multiverse
deb http://mirrors.163.com/ubuntu focal-backports main restricted universe multiverse
```



# 参考文档

https://developer.aliyun.com/mirror/ubuntu?spm=a2c6h.13651102.0.0.37d91b11TGGWc6

https://ywnz.com/linuxjc/1827.html

https://forum.ubuntu.org.cn/viewtopic.php?t=253103

https://wiki.ubuntu.com/Releases

