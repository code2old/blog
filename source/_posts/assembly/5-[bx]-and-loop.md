---
title: 汇编语言-[bx]和loop指令
data: 2020-08-01 20:50:00
updated: 2020-08-01 20:50:00
tags:
  - 汇编
categories: 
  - 汇编语言学习
---

### 第五章 [bx]和loop指令

[bx]和[0]有些类似，[0]表示内存单元，它的偏移地址是0。要完整的描述一个内存单元，需要两种信息：（1）内存单元的地址；（2）内存单元的长度（类型）。[bx]可以表示一个内存单元，它的偏移地址在bx中，比如以下指令：

```
mov ax, [bx]    # 将一个内存单元的内容送入ax，这个内存单元的长度为2（字单元），存放一个字，偏移地址在bx中，段地址在ds中。
```

后续课程中：用“()”表示一个寄存器或一个内存单元中的内容：

```
(ax)            # 表示ax中的内容
(al)            # 表示al中的内容
(20000H)        # 表示内存20000H单元的内容
((ds)*16+(bx))  # 表示ds中的内容为ADRR1，bx中的内容为ADRR2，ADRR1x16+ADRR2表示内存ADRR1:ADRR2单元的内容
```

<!-- more -->

#### [bx]

```
mov ax, [bx]
```
功能：bx中存放偏移地址EA，段地址SA默认在ds中，将SA:EA处的数据送入ax中。即：(ax)=((ds)*16+(bx))

```
mov [bx], ax
```
功能：bx中存放偏移地址EA，段地址SA默认在ds中，将ax处的数据送入SA:EA中。即：((ds)*16+(bx))=(ax)

#### Loop指令

loop指令的格式：loop标号，CPU执行loop指令时，要进行两步操作：（1）`(cx)=(cx)-1`；（2）判断cx中的值，不为零则转至标号处执行指令，如果为零则向下执行。**通常**用loop指令来实现循环功能，cx中存放循环次数。

#### 在Debug中跟踪用loop指令实现的循环程序

#### Debug和汇编编译器masm对指令的不同处理

在Debug中，`mov ax [0]`表示将ds:0处的数据送入ax中，在汇编源程序中，`mov ax, [0]`被编译器当作指令`mov ax, 0`处理。

源程序中可使用的方式：

```
mov ax, 2000h
mov ds, ax      ;段地址2000h送入ds
mov bx, 0       ;偏移地址0送入bx
mov al, [bx]    ;ds:bx单元中的数据送入al
```

或者

```
mov ax, 2000h
mov ds, ax
mov al, ds:[0]
```

在汇编源程序中，指令访问一个内存单元，则在指令中必须用`[...]`表示内存单元，如果在`[]`里用一个常量`idata`直接给出内存单元的偏移地址，则在`[]`的前面需要显示的给出段地址所在的寄存器，否则masm将把指令中的`[idata]`解释为`idata`

如果在`[]`里使用寄存器，比如bx，间接的给出内存单元的偏移地址，则段地址默认在ds中。当然，也可以显式的给出段地址所在的段寄存器。

#### loop和[bx]的联合应用

问题：计算ffff:0-ffff:b单元中的数据的和，结果存储在dx中。

思考：
* 运算后的结果是否会超出dx的表示范围？
* 是否能将ffff:0-ffff:b中的数据直接累加到dx中？
* 能否将ffff:0-ffff:b中的数据累加到dl中，并设置(dh)=0，从而实现累加到dx中的目标？
* 到底怎样将ffff:0-ffff:b中的8位数据累加到16位寄存器dx中？

方法：
* (dx)=(dx)+内存中的8位数据
* (dl)=(dl)+内存中的8位数据

第一种方法的问题是：运算对象的类型不匹配，第二种方法的问题是：运算结果有可能越界

```
assume cs:code
code segment

    mov ax, 0ffffh
    mov ds, ax

    mov bx, 0
    mov dx, 0
    mov cx, 0

s:  mov al, [bx]
    mov ah, 0
    add dx, ax
    inc bx
    loop s

    mov ax, 4c00h
    int 21h

code ends
end
```

#### 段前缀

指令`mov ax, [bx]`中内存单元的偏移地址由bx给出，而段地址默认在ds中，可以在访问内存单元的指令中显示的给出内存单元的段地址所在的段寄存器。比如：

```
mov ax, ds:[bx]
mov ax, cs:[bx]
mov ax, ss:[bx]
mov ax, es:[bx]
mov ax, cs:[0]
```

这些出现在访问内存单元指令中，用于显示指明内存单元段寄存器，在汇编语言中被称为**段前缀**

#### 一段安全的空间

在8086模式中，随意向一段内存空间写入内容是很危险的，因为这段空间中可能存放着重要的系统数据或者代码。所以在不能确定一段内存空间中是否存在着重要的数据或代码的时候，不能随意向其中写入数据。

#### 段前缀的使用

问题：将内存单元ffff:0-ffff:b中的数据拷贝到0:200-020b单元中

思考：
* 0:200-0:20b单元等同于0020:0-0020:b单元，它们描述的是同一段内存空间

```
assume cs:code
code segment

    mov ax, 0ffffh
    mov ds, ax
    mov ax, 0020h
    mov es, ax

    mov bx, 0
    mov cx 12

s:  mov dl, [bx]
    mov es:[bx], dl
    inc bx
    loop s

    mov ax, 4c00h
    int 21h

code ends
end
```

