---
title: 汇编语言-包含多个段的程序
data: 2020-08-11 20:50:00
updated: 2020-08-11 20:50:00
tags:
  - 汇编
categories: 
  - 汇编语言学习
---

### 第六章 包含多个段的程序

在操作系统的环境中，合法地通过操作系统取得的空间都是安全的，因为操作系统不会让一个程序所在的空间和其他程序以及系统自己的空间相冲突。操作系统允许的情况下，程序可以取得任意容量的空间。

程序取得所需空间的方法有两种：加载程序时为程序分配，程序执行过程中向系统申请。

#### 6.1 在代码段中使用数据

从规范的角度讲，是不能自己随便决定使用哪段空间的，应该让系统来分配空间。我们可以在程序中定义希望处理的数据，这些数据就会被编译、链接而作为程序的一部分被写入到可执行文件中，这样程序被加载到内存时，数据也被加载到内存中，此时就获得了空间。

end除了通知编译器程序结束外，还可以通知编译器程序的入口在什么地方。

可执行文件中的描述信息指明CPU的CS:IP指向程序的哪一条执行指令。程序来源于源程序中的汇编指令和定义的数据：描述信息则主要是编译、链接程序对源程序中相关伪指令进行处理所得到的信息。

程序框架：

```
assume cs:code
code segment
    ; 数据

start:
    ; 代码

code ends
end start
```

例如：

```
asssume cs:code
code segment

    dw 0123H, 0456H, 0789H, 0abcH, 0fedH, 0cbaH, 0987H

start:
    mov bx, 0
    mov ax, 0
    mov cx, 8

s:  add ax, cs:[bx]
    add bx, 2
    loop s

    mov ax, 4c00h
    int 21h

code ends
end start

```

#### 6.2 在代码段中使用栈

利用栈将程序中定义的数据逆序存放：

```
assume cs:codeseg
codeseg segment
    dw 0123H, 0456H, 0789H, 0abcH, 0defH, 0cbaH, 0987H
    dw 0, 0, 0, 0, 0, 0, 0, 0   ;用dw定义8个字型数据，
                                ;程序加载后，将取得8个字的内存空间，存放这8个数据
                                ;后面的程序中将这段空间当作栈来使用

start:  mov ax, cs
        mov ss, ax
        mov sp, 32              ;将设置栈顶ss:sp指向cs:32
        mov bx, 0
        mov cx, 8

s0:     push cs:[bx]            ;将代码段0-16单元中的8个字型数据以此入栈
        add bx, 2
        loop s0

        mov bx, 0
        mov cx, 8

s1:     pop cs:[bx]
        add bx, 2
        loop s1

        mov ax, 4c00h
        int 21h
codeseg ends
end start                       ;指明程序的入口地址在start处
```

#### 将数据、代码、栈放入不同的段

首先看一段代码

```
assume cs:code, ds:data, ss:stack
data segment
    dw 0123H, 0456H, 0789H, 0abcH, 0defH, 0cbaH, 0987H
data ends

stack segment
    dw 0, 0, 0, 0, 0, 0, 0, 0
stack ends

code segment
start:  mov ax, stack
        mov ss, ax
        mov sp, 16      ;设置栈顶ss:sp指向stack:16

        mov ax, data
        mov ds, ax      ;ds指向data段

        mov bx, 0       ;ds:bx指向data段的第一个单元
        mov cx, 8

s0:     push [bx]
        add bx, 2
        loop s1

        mov bx, 0
        mov cx, 8

s1:     pop [bx]
        add bx, 2
        loop s1

        mov ax, 4c00h
        int 21h
    
code ends
end start
```

定义多个段的方法：与之前定义代码段的方法没有区别，只是对于不同的段，要有不同的段名

对段地址的引用：在程序中，段名就相当于一个标号，代表了段地址。所以指令`mov ax, data`的含义就是将名称为`data`的段的段地址送入`ax`。一个段中的数据的段地址可由段名代表，偏移地址就要看它在段中的位置了。注意：8086CPU不允许将一个数值直接送入段寄存器，所以对段名的引用：如指令`mov ds, data`中的`data`将被编译器处理为一个表示段地址的数值。

```
mov ax, data
mov ds, ax
mov ax, ds:[6]
```

CPU到底如何处理我们定义的段中的内容，是当作指令执行，当作数据访问还是当作栈空间，完全是靠程序中具体的汇编指令，和汇编指令对CS:IP、SS:SP、DS等寄存器的设置来决定的。

