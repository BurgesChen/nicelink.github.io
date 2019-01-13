---
title: I.MX6 Linux Yocto开发环境搭建
tags: 
	- yocto
	- Linux
categories:
	- Linux开发环境
	
---

## 一、主机环境 
主机：Ubuntu16.04 LTS 64bit  

i.MX6 bsp infomation：  
Bsp version:fsl-yocto-L4.1.15_2.0.0-ga ;   
Linux Kernel version: 4.1 ;  
Yocto Project version: 2.1 ;  
CPU type: i.MX6 DualLite
## 二、Yocto简介
- Yocto：Yocto是一个开源社区它通过提供模版、工具和方法帮助开发者创建基于linux内核的定制系统，支持ARM, PPC, MIPS, x86 (32 & 64 bit)硬件体系架构。

- Poky：Poky有两个含义。第一个含义是用来构建Linux的构建系统，值得注意的该Poky仅仅是一个概念，而非一个实体：它包含了 BitBake工具、编译工具链、BSP、诸多程序包或层，可以认为Poky即是Yocto的本质；此外Poky还有另外一层意思，使用Poky系统得到的默认参考 Linux 发行版也叫Poky（当然，我们可以对此发行版随意命名），Poky的两个含义千万不能混淆。

- Metadata：元数据集，所谓元数据集就是发行版内各基本元素的描述与来源 
- Recipes：.bb/.bbappend文件，配方文件，描述了从哪获取软件源码，如何配置，如何编译。bbappend和bb的区别主要在于bbappend是基于bb的，功能是对相应的bb文件作补充和覆盖，有点类似于“重写”的概念。
- Class：.bbclass文件  

- Configuration：.conf文件，即配置文件，我们可以用它来改变构建方式。

- Layers：即各种meta-xxx目录，将Metadata按层进行分类，有助于项目的维护

- Bitbake：一个任务执行引擎，用来解析并执行Metadata

- Output：即各种输出的image。

==>注：以上介绍来源网络
## 三、I.MX6 Yocto环境搭建和编译步骤
&emsp; &emsp;为了使环境搭建更快，下载软件包速度加快，安装好ubuntu后需要去配置其更新软件源，配置为国内的软件源，如：网易源、阿里源等。具体的配置过程可参考网络教程：[《Ubuntu14.04更新源》](https://jingyan.baidu.com/article/7f41ecec1b7a2e593d095ce6.html)

1. ### Yocto环境安装
&emsp; &emsp;ubuntu安装和配置准备就绪后，开始搭建I.MX6的yocto环境（参考I.MX6官方文档），该yocto工程官方推荐使用ubuntu12.04或ubuntu14.04版本的OS。==安装步骤如下：==

（1）首先打开终端（快捷键：Ctrl+Alt+T），安装Yocto project 2.1的主机包软件，输入如下命令：  
$ sudo apt-get install gawk wget git-core diffstat unzip texinfo gcc-multilib 
build-essential chrpath socat libsdl1.2-dev

（2）安装yocto需要的软件包，输入如下命令：  
$ sudo apt-get install libsdl1.2-dev xterm sed cvs subversion coreutils texi2html docbook-utils python-pysqlite2 help2man make gcc g++ desktop-file-utils libgl1-mesa-dev libglu1-mesa-dev mercurial autoconf automake groff curl lzop asciidoc

（3）安装I.MX层的uboot工具，ubuntu12.04主机和ubuntu14.04主机输入的命令不一样，如下：
ubuntu12.04主机安装命令：sudo apt-get install uboot-mkimage
ubuntu14.04主机安装命令：sudo apt-get install u-boot-tools

（4）然后开始建立repo工具，Repo是在Git之上构建的工具，它使得管理包含多个存储库的项目变得更加容易，而这些存储库不需要位于同一服务器上。Repo很好地补充了Yocto项目的分层特性，使得用户可以更容易地将自己的层添加到BSP中。依次输入如下命令：  
$ mkdir ~/bin  
$ curl http://commondatastorage.googleapis.com/git-repo-downloads/repo > ~/bin/repo  
$ chmod a+x ~/bin/repo
$ export PATH=~/bin:$PATH

<font color=red face="黑体">上述步骤的注意事项</font>：  
- 输入上面第二条命令的时候是会提示失败的，因为那个http网址是google相关的网址，国内是访问不了的，需要使用VPN代理服务器访问，再输入命令：   
&emsp; $ curlhttp://commondatastorage.googleapis.com/git-repo-downloads/repo > ~/bin/repo 

- <font color=red face="黑体">注意</font>：如果没有VPN，可以使用国内清华源的repo下载网址替代，参考[Git Repo 镜像使用帮助](https://mirrors.tuna.tsinghua.edu.cn/help/git-repo/)，命令如下：  
&emsp; $ curl https://mirrors.tuna.tsinghua.edu.cn/git/git-repo -o ~/bin/repo  
下一步还要导入repo的环境变量，这样reposync才会去清华源下载相关软件包，命令如下：
&emsp; $ export REPO_URL='https://mirrors.tuna.tsinghua.edu.cn/git/git-repo/'

（5）然后开始设置Yocto工程的Git配置，输入如下命令：  
$ git config --global user.name "Your Name"  
$ git config --global user.email "Your Email"  
$ git config --list  

（6）创建bsp代码和工具的存放目录，在work目录（我这里存放bsp的目录是work，可以自己随意调整存放目录）下创建一个fsl-release-bsp目录，然后进入该目录：
$ mkdir /work/fsl-release-bsp
$ cd /work/fsl-release-bsp

（7）然后使用repo同步命令去更新和下载代码，同步和下载的速度视VPN的网速而定，命令如下：  
repo init -u git://git.freescale.com/imx/fsl-arm-yocto-bsp.git -b imx-4.1-krogoth
repo sync 

### 2.编译yocto工程
（1）首先是构建交叉编译环境，Freescale官方提供了一个 fsl-setup-release.sh来设置交叉编译环境，需要指定的特定机器的名称以及所需的图形后端，脚本设置指定机器和后端的目录和配置文件。  
MACHINE选项用于指定板子和	CPU名称，最新的机器名称如下：  
• imx6qpsabreauto  
• imx6qpsabresd  
• imx6ulevk  
• imx6ull14x14evk  
• imx6ull9x9evk  
• imx6dlsabreauto  
• imx6dlsabresd  
• imx6qsabreauto  
• imx6qsabresd  
• imx6slevk  
• imx6solosabreauto  
• imx6solosabresd  
• imx6sxsabresd  
• imx6sxsabreauto  
• imx7dsabresd  

&emsp; &emsp;DISTRO配置选项，用于指定后端图像配置，如果你没有指定DISTRO，那在最后用-e 选项指定也可以，可以支持的后端图像有：  
• fsl-imx-x11 - Only X11 graphics  
• fsl-imx-wayland - Wayland weston graphics  
• fsl-imx-xwayland - Wayland graphics and X11. X11 applications using EGL are not supported  
• fsl-imx-fb - Frame Buffer graphics - no X11 or Wayland  

-b 选项用于指定构建目录

（2）配置命令如下

$ DISTRO=fsl-imx-wayland MACHINE=imx6dlsabresd source fsl-setup-release.sh -b imx6dlsabresd-build-wayland

（3）选择要编译的镜像类型
可选的镜像类型如下图所示：  
![image-type](image-type.png)  
我这里选择的是 fsl-image-gui，编译命令如下：  
$ bitbake fsl-image-gui

<font color=red face="黑体">注意</font>：：如果想要离线编译（无网络情况下），可以先选择先下载编译项所需要的pack再去编译，fetchall命令如下：   
$ bitbake fsl-image-gui -c fetchall  
&emsp; &emsp;然后就静静的等待其编译完成，编译需要漫长的等待，如果出错，查看错误后继续尝试，最好是用VPN网络。  
&emsp; &emsp;如果想要编译速度更快，可以修改/fsl-release-bsp/build（创建的构建目录）/conf/local.conf文件，添加如下内容，数字代表同时执行的编译任务数目： BB_NUMBER_THREADS = '4'  
PARALLEL_MAKE = '-j 4'

&emsp; &emsp;重新打开终端时，进入build，需要先导入环境变量才能去编译，输入如下命令：  
$ source setup-environment imx6qsabresd-build-wayland/  

### 3.安装交叉编译工具链  
（1）输入编译工具链命令：  
$ bitbake meta-toolchain  

（2）等待编译完成后，进行到 tmp/deploy/sdk目录运行脚本安装，输入如下命令：  
$ cd tmp/deploy/sdk  
$ ./fsl-imx-wayland-glibc-x86_64-meta-toolchain-cortexa9hf-neon-toolchain-4.1.15-2.1.0.sh  

（3）会出现提示如下，选择安装目录（默认），敲回车即可。  
Enter target directory for SDK (default: /opt/fsl-imx-wayland/4.1.15-2.1.0): （**回车，选择默认安装路径**）  
The directory "/opt/fsl-imx-fb/4.1.15-1.2.0" already contains a SDK for this architecture.  
If you continue, existing files will be overwritten! Proceed[y/N]? y  

（4）安装完成后，按照提示导入环境变量，输入如下命令：

$ . /opt/fsl-imx-wayland/4.1.15-2.1.0/environment-setup-cortexa9hf-neon-poky-linux-gnueabi  

&emsp; &emsp;导入环境变量后，输入env命令可以看到如下项：  
CC=arm-poky-linux-gnueabi-gcc  -march=armv7-a -mfpu=neon  -mfloat-abi=hard -mcpu=cortex-a9 --sysroot=/opt/fsl-imx-wayland/4.1.15-2.1.0/sysroots/cortexa9hf-neon-poky-linux-gnueabi  
&emsp; &emsp;后面我们就可以使用$CC来编译自己的程序放在ARM机器上运行。

（5）helloworld程序测试
- 编译命令：$CC hello.c -o hello  
- 编译完后，可以使用file命令查看文件属性：  
$ file hello  
属性显示如下：  
hello: ELF 32-bit LSB executable, ARM, EABI5 version 1 (SYSV), dynamically linked, interpreter /lib/ld-linux-armhf.so.3, for GNU/Linux 2.6.32, BuildID[sha1]=e03e10e95a24ad4f1c23c92962131db3d71f652b, not stripped  

## 三、I.MX6 Yocto环境编译总结
&emsp; &emsp;编译过程中会出现各种各样的错误，不要放弃，上网查找原因和解决办法，很多错误是由于网络问题或者网址变更，没有将pack下载下来。一些错误总结如下：  

（1）do_fetch failed错误  
&emsp; &emsp;在git fetch失败的时候，我们可以查看对应包的*.bb文件中的SRC_URI项（不同的pack源码下载网址），可以去那个网址手动下载，或者git clone下来，放到指定目录，然后去修改*.bb文件中的SRC_URI项。
具体操作步骤可以参考下网址文章：  
https://blog.csdn.net/groundhappy/article/details/55046166
https://blog.csdn.net/sy373466062/article/details/50363537


(2). 单独编译指令  
- 配置menuconfig指令：  
&emsp; &emsp;bitbake -c menuconfig -v linux-imx   
- 单独编译kernel、模块、设备树：  
&emsp; &emsp;bitbake -c compile -f -v linux-imx  
&emsp; &emsp;bitbake linux-imx -c compile_kernelmodules -f -v    
&emsp; &emsp;bitbake -c deploy -f -v linux-imx  

- 若要编译文件系统则用下面总指令：  
&emsp; &emsp;bitbake core-image-minimal  
&emsp; &emsp;bitbake -c compile -f -v u-boot-imx  
&emsp; &emsp;bitbake -c deploy -f -v u-boot-imx  

## 四、参考内容
- 《i.MX_Yocto_Project_User's_Guide.pdf》
