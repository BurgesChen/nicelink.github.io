---
title: iMX6 QT5交叉编译环境搭建for Yocto
date: 2019-01-14 22:00:03
tags: 
	- yocto
	- Linux
	- Qt
categories:
	- Linux开发环境
	- Qt开发

---

## 一、主机环境

Ubuntu版本：14.04.5 64bit  
Qt Version: 5.6.2  
CPU: I.MX6 DualLite    
I.MX6 bsp infomation：  
Bsp version:fsl-yocto-L4.1.15_2.0.0-ga ;   
Yocto Project version: 2.1 ;  
Linux Kernel version: 4.1 ;  
U-Boot version: 2016.03-r0  

## 二、名词术语解释

序号 | 名词 | 解释   
---|---|---  
1 | <Build-DIR\> | yocto构建目录路径，配置DISTRO=fsl-imx-wayland MACHINE=imx6dlsabresd source fsl-setup-release.sh -b imx6dlsabresd-build-wayland命令中，-b选项所指定的目录  
2 | meta-toolchain-qt5 | yocto中的qt5交叉编译工具包  
3 | <user\> | Ubuntu系统中的用户名

## 三、QT交叉编译环境搭建  
&emsp; &emsp;要生成可以在ARM-Linux平台上运行的QT程序，就必须生成QT for ARM相关的编译工具链去搭建QT的开发环境，QT的交叉编译环境最主要的就是要用到QT的交叉编译工具链，交叉编译工具链的生成方式有以下两种：  
（1）使用arm交叉编译工具链来编译QT的源码，生成对应ARM平台的QT编译器和构建工具。  
（2）使用Yocto部署下的qt交叉编译工具链的pack,生成对应ARM平台的QT编译器和构建工具。  

### 1.Yocto下编译部署QT交叉编译工具链
&emsp; &emsp;当我们建立好了Yocto for IMX6的开发环境时，可以直接编译和部署meta-toolchain-qt5这个包，步骤如下：
（1）打开终端，启动yocto的构建环境，以下命令请根据构建环境目录名称不同自行修改：  
$ cd fsl-release-bsp/  
$ source setup-environment <Build-DIR>/  

（2）编译和部署QT5交叉编译工具链，命令如下：  
$ bitbake meta-toolchain-qt5  

（3）安装QT5交叉编译工具链，安装脚本生成在Yocto构建目录<Build-DIR>/tmp/deploy/sdk下：  
$ cd <Build-DIR>/tmp/deploy/sdk  
$ ./tmp/deploy/sdk/fsl-imx-wayland-glibc-x86_64-meta-toolchain-qt5-cortexa9hf-neon-toolchain-4.1.15-2.1.0.sh  
&emsp; &emsp;安装过程中会提示你指定安装目录，本文设置的是/home/<user>/opt/imx-meta-toolchain-qt5/fsl-imx-wayland/4.1.15-2.1.0/

（4）安装完成后，可以使用“source”或“. /” 命令来导入环境变量：  
$ . /home/nice/opt/imx-meta-toolchain-qt5/fsl-imx-wayland/4.1.15-2.1.0/environment-setup-cortexa9hf-neon-poky-linux-gnueabi

### 2.下载和安装QT5
&emsp; &emsp;本文使用的yocto版本中包含的QT5交叉编译工具链是基于qt5.6.2版本的，我们这里也选择5.6.2版本。为了方便在Ubuntu上调试运行，我们这里下载的是QT SDK for linux，包含了<font color=red face="courier new">Qt库、Qt Creator IDE和Qt工具</font>，当然如果不想在Ubuntu PC上运行QT程序，直接下载Qt Creator即可。  

（1）进入QT下载页去下载：http://download.qt.io/official_releases/qt/5.6/5.6.2/  
![qt5_download](qt5_download.png)

（2）下载完成后，给该文件加上可执行权限，然后运行安装即可：  
$ chmod +x qt-opensource-linux-x64-5.6.2.run  
$ sudo ./qt-opensource-linux-x64-5.6.2.run  

- 开始安装之后，按照提示操作一路Next，然后选择安装路径，本文使用的路径为：/home/<user>/opt/Qt5.6.2  
![qt5_install](qt5_install.png)  

- 然后会出现选择安装的组件界面，默认即可，Next，等待安装完成。  

（3）<font color=red face="courier new">**安装完成后,去编辑/home/user/Qt5.6.2/Tools/QtCreator/bin/qtcreator.sh，在文件开头加上一个QT交叉编译工具的环境变量导入命令**</font>：source /home/nice/opt/imx-meta-toolchain-qt5/fsl-imx-wayland/4.1.15-2.1.0/environment-setup-cortexa9hf-neon-poky-linux-gnueabi，去下图所示：（注意：此步骤非常重要！！！）  
![sh](sh.png)

### 3.配置Qt Creator  
#### 3.1配置Qt for ARM交叉编译环境  
&emsp; &emsp;打开Qt Creator后，将QT5的交叉编译工具链添加进来使用，步骤如下：

（1）点击Tools -> Options，选中Build & Run 一栏进行设置，首先切换到Compiler，点击Add -> GCC 去添加一个GCC编译器，Compiler path选择: /home/nice/opt/imx-meta-toolchain-qt5/fsl-imx-wayland/4.1.15-2.1.0/sysroots/x86_64-pokysdk-linux/usr/bin/arm-poky-linux-gnueabi/arm-poky-linux-gnueabi-g++，然后点击Apply，如下图所示：  
![add_compilers](add_compilers.png)  

（2）然后切换到Qt Version，去添加一个Qt版本，选择生成的QT交叉编译工具链的qmake路径：/home/nice/opt/imx-meta-toolchain-qt5/fsl-imx-wayland/4.1.15-2.1.0/sysroots/x86_64-pokysdk-linux/usr/bin/qt5/qmake，然后点击Apply，如下图所示：
![add_qmake](add_qmake.png)  

&emsp; &emsp;该qmake的详细信息如下图所示，<font color=green face="courier new">QMAKE_SPEC的值默认是linux-g++ </font>： 
![qmake_spec](qmake_spec.png)  

（3）然后切换到Kits，点击Add ，填写Name，Device type，Sysroot选择路径是：/home/nice/opt/imx-meta-toolchain-qt5/fsl-imx-wayland/4.1.15-2.1.0/sysroots/cortexa9hf-neon-poky-linux-gnueabi，Compiler选择前面步骤（1）添加的IMX-QT5-GCC，Qt Version选择前面步骤（2）添加的IMX-Qt5.6.2(qt5)，点击OK。  
![add_kits](add_kits.png)

#### 3.2创建测试工程并且编译
（1）首先点击File -> New File or Project，然后出现如下界面，一次选择Application 和 Qt Widgets Application，再点击Choose。  
![new_app](new_app.png)  

（2）然后输入工程名和工程路径，如下图所示：  
![project_name](project_name.png)  

（3）然后选择Kits，一个是构建linux桌面应用程序的（默认安装qt时就有），另一个是我们后面添加的IMX-QT-kit，我们用于构建ARM上运行的Qt程序，这里我们将两个都选择上。  
![choose_kits](choose_kits.png)  

（4）然后设置生成的代码文件和基类相关信息，我们这里直接使用默认值。  
![class](class.png)  

（5）最后选择是否使用版本管理，我们不使用，选择None,然后点击Finish。  
![git_ver](git_ver.png)  

（6）工程创建成功，可以在左侧看到源文件，如下图所示：  
![src](src.png)  

（7）然后双击打开mainwindow.ui，在左边的控件窗口拖动一个Label到界面上，双击编辑，输入“Qt Creator，Test!”  
![test](test.png)  

（8）然后先选择桌面版本的，编译运行看看结果。  
![result0](result0.png)  

![result1](result1.png)  

（9）开始编译ARM版本，左下角选择IMX-QT-Kit，然后点击build，如下图所示：  
![imx_qt](imx_qt.png)  

（10）编译后提示错误，没有找到相关组件，错误信息如下：
![err0](err0.png)  

![err0](err0.png)  
（11）查看qmake生成的Makefile，发现CC和CXX使用的是gcc和g++，编译命令-spec默认使用的是linux-g++，如下图所示，猜测应该是平台和编译相关的描述文件qmake.conf（QMAKESPEC环境变量决定使用哪个qmake.conf）没有设置好。  

![qmake_conf](qmake_conf.png)  

（12）为了验证问题，在qtcreator.sh脚本导入环境变量后，加入env命令来打印环境变量，查看环境变量发现QMAKESPEC=/home/nice/opt/imx-meta-toolchain-qt5/fsl-imx-wayland/4.1.15-2.1.0/sysroots/cortexa9hf-neon-poky-linux-gnueabi/usr/lib/qt5/mkspecs/linux-oe-g++，可以看出环境变量设置值linux-oe-g++和编译命令 “-spec linux-g++”不一样，故点击左侧Project选项卡，修改编译命令行，如下图所示：
- 修改前  
![modify_f](modify_f.png)  

- 修改后  
![modify_r](modify_r.png)

（13）修改完毕后，先执行clean，然后再点击build，大功告成，不会再有报错；将buile-test-IMX_QT_Kit-release目录中的test程序拷贝到IMX6机器上运行即可（注意：运行成功的条件是imx6 上的linux系统中的文件系统有相关qt运行环境支持，在yocto环境中运行bitbake fsl-image-qt5可得到相应的文件系统）。
QT程序运行时需要注意的是后面需要带有平台图形相关的参数，指定使用的平台选项如下：  
![arm_opt](arm_opt.png)  

- 输入的运行test程序的命令如下：
$ ./test -platform wayland-egl 

- IMX6机器上运行的结果如下图所示：  
![arm_result](arm_result.png)  

## 四、问题总结
1.运行QtCreator，出界面后，报一大堆错误，文件没有权限等；  
<font color=red face="courier new">错误原因</font>： 没有权限访问相关文件。

<font color=green face="courier new">解决办法</font>：  使用root权限打开QtCreator。

2.编译Qt for arm程序时，报错，找不到相关组件。  
<font color=red face="courier new">错误原因</font>：多半是环境设置的问题，相关环境变量设置不正确。

<font color=green face="courier new">解决办法</font>： 检查Makefile，看是否使用了正确的GCC编译器，确保使用了Qt for ARM交叉编译相关的GCC；检查QMAKESPEC环境变量的值，是否使用了Qt for ARM交叉编译目录mkspec下的正确qmake.conf配置。

