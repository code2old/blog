---
title: 汇编语言-基础知识
tags:
  - 汇编
categories:  
  - 汇编语言学习
---

### 第一章 基础知识

#### 1.1 机器语言

机器语言是机器指令的集合。机器指令展开来讲就是一台机器可以正确执行的命令。电子计算机的机器指令是一些列二进制数字。计算机将之转变为一系列高低电平，以使计算机的电子器件受到驱动，进行运算。

#### 1.2 汇编语言的产生

汇编语言的主体是汇编指令。汇编指令和机器指令的差别在于指令的表示方法上。汇编指令则是机器指令便于记忆的书写格式。
```
操作：寄存器BX的内容送到AX中
机器指令：1000100111011000
汇编指令：mov ax, bx
```
将汇编指令转换成机器指令的翻译程序就被称为编译器。

#### 1.3 汇编语言的组成

汇编语言由以下3类指令组成：
* 汇编指令：机器码的助记符，有对应的机器码
* 伪指令：没有对应的机器码，由编译器执行，计算机并不执行
* 其他符号：如+、-、*、/等，由编译器识别，没有对应的机器码。
汇编语言的核心是机器指令，它决定了汇编语言的特性

#### 1.4 存储器

CPU是计算机的核心部件，它控制整个计算机的运作并进行运算。指令和数据在存储器中存放。学习汇编首先要了解CPU是如何从内存中读取信息，以及向内存中写入信息的。

#### 1.5 指令和数据

指令和数据都是应用上的概念。在内存和磁盘中指令和数据没有任何区别，都是二进制信息。

#### 1.6 存储单元

存储器被划分为若干个存储单元，每个存储单元从0开始顺序编号。电子计算机的最小信息单位是bit，微机存储器的容量是以字节为最小单位来计算的。

#### 1.7 CPU对存储器的读写

CPU要想进行数据的读写，必须和外部器件进行3类信息交换：
* 存储单元的地址（地址信息）
* 器件的选择，读或写的命令（控制信息）
* 读或写的书写（数据信息）
计算机中专门连接CPU和其他芯片的导线被称为总线，从逻辑上将分为3类：地址总线、控制总线和数据总线

#### 1.8 地址总线

CPU是通过地址总线来指定存储单元的。地址总线上能传送多少个不同的信息，CPU就可以对多少个存储单元进行寻址。一个CPU有N跟地址总线，则这个CPU的地址总线宽度为N。这样CPU最多可以对2^N个内存单元进行寻址

#### 1.9 数据总线

CPU通过数据总线与其它器件进行数据交互。总线宽度决定了CPU与外界的数据传送速度。8088CPU数据总线宽度为8，一次可传送一个字节，8086CPU的数据总线宽度为16，一次可传送2个字节。

#### 1.10 控制总线

CPU通过控制总线对外部器件进行控制，控制总线的宽度决定了CPU对外部器件的控制能力。

#### 1.11 内存地址空间概述

##### 1.11.1 主板

##### 1.11.2 接口卡

CPU通过总线向接口卡发送命令，接口卡根据CPU的命令控制外设进行工作

#### 1.12 各类存储器芯片

存储器从读写属性上分为两类：随机存储器（RAM）和只读存储器（ROM）。随机存储器可读可写，但必须带电存储，断电后存储内容丢失；只读存储器只能读取不能写，断电后内容不丢失。存储器从功能和连接上又分为以下几类：
* 随机存储器
* 装有BIOS（Basic Input/Output System）的ROM
* 接口卡上的RAM

#### 1.13 内存地址空间

所有物理存储器被看作一个由若干存储单元组成的逻辑存储器，每个物理存储器在这个逻辑存储器中占有一个地址段，即一段地址空间。8086CPU的地址总线宽度为20，可以定位2^20(1M)的内存单元，80386地址总线宽度为32，最大寻址空间为4G。8086地址空间分配基本情况如下：

```
00000 |————————————————————|
      |   主存储器地址空间  |
      |      (RAM)         |
9FFFF |____________________|
A0000 |                    |
      |    显存地址空间     |
BFFFF |____________________|
C0000 |                    |
      |   各类ROM地址空间   | 
FFFFF |____________________|   
```

#### 小结

* 汇编指令是机器指令的助记符，同机器指令一一对应
* 每一种CPU都有自己的汇编指令集
* CPU直接使用的信息存放在存储器中
* 在存储中指令和数据没有任何区别，都是二进制信息
* 存储单元从0开始顺序编号
* 一个存储单元可以存储8个bit，即8位二进制数
* 1B=8b 1KB=1024B 1MB=1024KB 1GB=1024MB
* 每一个CPU芯片都有许多管脚，这些管脚和总线相连。一个CPU可以引出三种总线的宽度标志了这个CPU不同方面的性能：
    * 地址总线的宽度决定了CPU的寻址能力
    * 数据总线的宽度决定了CPU与其它器件进行数据传输的一次数据传输量
    * 控制总线的宽度决定了CPU对系统中其它器件的控制能力

对CPU来讲，系统中的所有存储器中的存储单元都处于一个统一的逻辑存储器中，它的容量受CPU寻址能力的限制。这个逻辑存储器即是我们所说的内存地址空间。