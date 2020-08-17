---
title: 汇编语言-call和ret指令
data: 2020-08-15 22:44:00
updated: 2020-08-17 21:44:00
tags:
  - 汇编
categories: 
  - 汇编语言学习
---

### 第十章 call和ret指令

call和ret指令都是转移指令，它们都修改IP，或同时修改CS和IP。它们经常被共同用来实现子程序的设计。

#### ret和retf

ret指令用栈中的数据，修改IP的内容，从而实现近转移
retf指令用栈中的数据，修改CS和IP的内容，从而实现远转移 

CPU执行ret指令时，进行下面两步操作：

* (IP)=((SS)*16+(SP))
* (SP)=(SP)+2

CPU执行retf指令时，进行下面四步操作：

* (IP)=((SS)*16+(SP))
* (SP)=(SP)+2
* (CS)=((SS)*16+(SP))
* (SP)=(SP)+2

可以看出，如果用汇编语法来解释ret和retf指令，则：
CPU执行ret指令时，相当于执行：

```
pop IP
```

CPU执行retf指令时，相当于执行：

```
pop IP
pop CS
```

下面的程序中，ret指令执行后，(IP)=0,CS:IP执行代码段的第一条指令

```
assume cs:code
stack segment
    db 16 dup (0)
stack ends

code segment
    mov ax, 4c00h
    int 21h

start:
    mov ax, stack
    mov ss, ax
    mov sp, 16
    mov ax, 0
    push ax
    mov bx, 0
    ret
code ends
end start
```

下面的程序中，retf指令执行后，CS:IP指向代码段的第一条指令

```
assume cs:code
stack segment
    db 16 dup (0)
stack ends

code segment
    mov ax, 4c00h
    int 21h

start:
    mov ax, stack
    mov ss, ax
    mov sp, 16

    mov ax, 0
    push cs
    push ax
    mov bx, 0
    retf
code ends
end start
```

#### call指令

CPU执行call指令时，进行两步操作：

* 将当前的IP或CS和IP压入栈中
* 转移

call指令不能实现短转移，除此之外，call指令实现转移的方法和jmp指令的原理相同

#### 依据位移进行转移的call指令

call 标号(将当前的IP压栈后，转到标号处执行指令)

CPU执行此种格式的call指令时，进行如下操作：

* (SP)=(SP)-2
  ((SS)*16+(SP))=(IP)
* (IP)=(IP)+16位位移

16位位移=“标号”处的地址-call指令后的第一字节的地址
16位位移的范围为-32768~32767，用补码表示
16位位移的编译程序在编译时算出

如果用汇编语法解释此种格式的call指令，则相当于

```
push IP
jmp near ptr 标号
```

#### 转移的目的地址在指令中的call指令

指令`call far ptr标号`实现的是段间转移，此种call指令操作如下：

* (SP)=(SP)-2
  ((SS)*16+(SP))=(CS)
  (SP)=(SP)-2
  ((SS)*16+(SP))=(IP)
* (CS)=标号所在段的段地址
  (IP)=标号所在段中的偏移地址

如果用汇编语法解释此种格式的call指令，则相当于

```
push CS
push IP
jmp far ptr 标号
```

#### 转移地址的寄存器中的call指令

指令格式：`call 16为寄存器`
功能：

(SP)=(SP)-2
((SS)*16+(SP))=(IP)
(IP)=(16位寄存器)

如果用汇编语法解释此种格式的call指令，则相当于

```
push IP
jmp 16位寄存器
```

#### 转移指令在内存中的call指令

转移地址在内存中的call指令有两种格式：
格式一

```
call word ptr 内存单元地址
```

如果用汇编语法解释此种格式的call指令，则相当于

```
push IP
jmp word ptr 内存单元地址
```

比如下面的指令：

```
mov sp, 10h
mov ax, 0123h
mov ds[0], ax
call word ptr ds:[0]
```
执行后，(IP)=0123, (SP)=0EH


格式二

```
call dword ptr 内存单元地址
```

如果用汇编语法解释此种格式的call指令，则相当于

```
push CS
push IP
jmp dword ptr 内存单元地址
```

比如下面的指令：

```
mov sp, 10h
mov ax, 0123h
mov ds:[0], ax
mov word ptr ds:[2], 0
call dword ptr ds:[0]
```
执行后，(CS)=0，(IP)=0123H，(SP)=0CH

#### call和ret配合使用

下面程序返回前，bx中的值是多少

```
assume cs:code
code segment
start:
    mov ax, 1
    mov cx, 3
    call s
    mov bx, ax  ;(bx)=?
    mov ax, 4c00h
    int 21h
s:
    add ax, ax
    loop s
    ret

code ends
end start
```

#### mul指令

mul是乘法指令，使用mul做乘法的时候：

* 两个想乘的数，要么都是8位，要是都是16位。如果是8位，一个默认放在AH中，另一个放在8位寄存器或内存单元中；如果是16位，一个默认在AX中，另一个放在16位寄存器或内存单元中
* 结果：如果是8位乘法，结果默认放在AX中；如果是16位乘法，结果高位默认在DX中，低位在AX中

格式如下：

```
mul reg
mul 内存单元
```

#### 模块化程序设计

call和ret指令共同支持了汇编语言中的模块化设计。

#### 参数和结果传递的问题

讨论参数和返回值传递的问题，实际上是探讨如何存储子程序需要的参数和产生的返回值。用寄存器来存储参数和结果是最常使用的方法，对于存放参数和存放结果的寄存器，调用者和子程序的读写操作恰恰相反：调用者将参数送入参数寄存器，从结果寄存器中取得返回值；子程序从参数寄存器中取得参数，将返回值送入结果寄存器

#### 批量数据的传递

批量传参和结果时，可以将批量数据存放到内存中，然后将它们所在内存空间的首地址放在寄存器中，传递给需要的子程序。对于具有批量数据的返回结果，也可用同样的方法。

#### 寄存器冲突问题

在子程序的开始将子程序中所有用到的寄存器中的内容保持起来，在子程序返回前再回复。可以用栈来保持寄存器的内容。

所以编写子程序的标志框架如下：

```
子程序开始:
    子程序中使用的寄存器入栈
    子程序内容
    子程序中使用的寄存器出栈
    返回(ret/retf)
```