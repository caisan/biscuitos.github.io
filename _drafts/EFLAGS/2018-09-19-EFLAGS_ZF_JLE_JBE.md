---
layout: post
title:  "JLE/JBE  ZF 变化引起的跳转"
date:   2018-09-19 19:23:30 +0800
categories: [MMU]
excerpt: JLE/JBE  ZF 变化引起的跳转.
tags:
  - EFLAGS
  - ZF
---

## 原理

Intel X86 提供了 JLE/JBE 指令，该指令用于基于 ZF 标志的跳转。如果 ZF 置位，
则 JLE/JBE跳转

{% highlight ruby %}
JBE    ------ 小于或等于的时候跳转，用于无符号
JLE    ------ 小于或等于的时候跳转，用于有符号
{% endhighlight %}

## 实践

BiscuitOS 提供了 JLE/JBE 相关的实例代码，开发者可以使用如下命令：

首先，开发者先准备 BiscuitOS 系统，内核版本 linux 1.0.1.2。开发可以参照文档
构建 BiscuitOS 调试环境：

{% highlight ruby %}
https://biscuitos.github.io/blog/Linux1.0.1.2_ext2fs_Usermanual/
{% endhighlight %}


接着，开发者配置内核，使用如下命令：

{% highlight ruby %}
cd BiscuitOS
make clean
make update
make linux_1_0_1_2_ext2_defconfig
make
cd BiscuitOS/kernel/linux_1.0.1.2/
make clean
make menuconfig
{% endhighlight %}

由于 BiscuitOS 的内核使用 Kbuild 构建起来的，在执行完 make menuconfig 之后，
系统会弹出内核配置的界面，开发者根据如下步骤进行配置：

![Menuconfig](https://raw.githubusercontent.com/EmulateSpace/PictureSet/master/BiscuitOS/kernel/MMU000003.png)

选择 **kernel hacking**，回车

![Menuconfig1](https://raw.githubusercontent.com/EmulateSpace/PictureSet/master/BiscuitOS/kernel/MMU000004.png)

选择 **Demo Code for variable subsystem mechanism**, 回车

![Menuconfig2](https://raw.githubusercontent.com/EmulateSpace/PictureSet/master/BiscuitOS/kernel/MMU000005.png)

选择 **MMU (Memory Manager Unit) on X86 Architecture**, 回车

![Menuconfig3](https://raw.githubusercontent.com/EmulateSpace/PictureSet/master/BiscuitOS/kernel/MMU000006.png)

选择 **Data storage： Main  Memory, Buffer, Cache**, 回车

![Menuconfig4](https://raw.githubusercontent.com/EmulateSpace/PictureSet/master/BiscuitOS/kernel/MMU000007.png)

选择 **Register: X86 Common Register mechanism**, 回车

![Menuconfig5](https://raw.githubusercontent.com/EmulateSpace/PictureSet/master/BiscuitOS/kernel/MMU000008.png)

选择 **EFLAGS： Current status register of processor**, 回车

选择 **ZF Zero flag (bit 6)**.

选择 **JLE    Jump short if less or equal**

或者

选择 **JBE  jump short if below or equal**

![Menuconfig6](https://raw.githubusercontent.com/EmulateSpace/PictureSet/master/BiscuitOS/kernel/MMU000334.png)

运行实例代码，使用如下代码：

{% highlight ruby %}
cd BiscuitOS/kernel/linux_1.0.1.2/
make 
make start
{% endhighlight %}

![Menuconfig7](https://raw.githubusercontent.com/EmulateSpace/PictureSet/master/BiscuitOS/kernel/MMU000335.png)

## 源码分析

源码位置：

{% highlight ruby %}
BiscuitOS/kernel/linux_1.0.1.2/tools/demo/mmu/storage/register/EFLAGS/eflags.c
{% endhighlight %}

![Menuconfig8](https://raw.githubusercontent.com/EmulateSpace/PictureSet/master/BiscuitOS/kernel/MMU000336.png)

源码如上图，将立即数 0x10 存储到 BL 寄存器中，将立即数 0x10 存储到 AL 寄存
器中。调用 CMP 指令，如果对比的结果相等，则 ZF 置位，程序跳转到 ZF_SY 分支，
并将立即数 1 存储到寄存器 DX 中；反之跳转到 ZF_CY 分支执行，并将立即数 0 存
储到寄存器 DX 中。最后将 DX 寄存器中的值存储到 ZF 变量里，并将 AX 寄存器的值
存储到 AX 变量里。

#### 运行结果如下：

![Menuconfig9](https://raw.githubusercontent.com/EmulateSpace/PictureSet/master/BiscuitOS/kernel/MMU000337.png)

#### 运行分析：

当 CONFIG_DEBUG_ZF_JLE 宏定义，则将立即数 0x10 存储到寄存器 AL 中，CMP 指令
执行之后，AL 寄存器中的值等于 BL 寄存器中的值，所以 ZF 置位，立即数 1 被存储
到 DX 寄存器中，最终 DX 寄存器的值 1 存储到 ZF 变量中。

当 CONFIG_DEBUG_ZF_JBE 宏定义，则将立即数 0x10 存储到寄存器 AL 中，CMP 指令执
行之后，AL 寄存器中的值等于 BL 寄存器中的值，所以 ZF 置位，立即数 1 被存储到 
DX 寄存器中，最终 DX 寄存器的值 1 存储到 ZF 变量中。

#### 实践结论：

ZF 置位时 JLE 进行短跳

ZF 置位时 JBE 进行短跳

## 运用场景分析

## 附录

[1. JLE/JBE 指令: Intel Architectures Software Developer's Manual: Combined Volumes: 2 Instruction Set Reference,A-Z-- Chapter 3 Instruction Set Reference,A-L: 3.2 Instruction(A-L) : JLE/JBE -- Jump](https://software.intel.com/en-us/articles/intel-sdm)

[2. Intel Architectures Software Developer's Manual](https://github.com/BiscuitOS/Documentation/blob/master/Datasheet/Intel-IA32_DevelopmentManual.pdf)
