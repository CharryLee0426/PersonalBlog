---
title: Lichen_Kernel
date: 2021-06-19
tag: 自制操作系统
categories: 计算机技术
top: true
mathjax: false
---

## Lichen_Kernel

> 学完了操作系统，老师讲的非常一般，学校配套的教材也写的不是特别好。我认为学习操作系统最好的方法就是自己写一套自己的简单的内核。所以我自己给自己布置了一套作业，就是跟随教程和自己的思考写一套自己的内核 `Lichen_Kernel`。

### 0. 前期准备

1. 下载文件 `30天自制操作系统文件CD`。
2. 下载二进制编辑器 `BZ.exe`，[点击此处下载](http://www.vcraft.jp/soft/bz.html)。
3. 开源虚拟机软件`VisualBox`下载，前往[清华源](https://tuna.moe)下载。
4. 设置系统变量，设置前将中文路径替换成英文的，方便以后使用原作者自己编写的工具。

### 1. 初步认识

这里第一个需要注意的点是如何让`VisualBox`运行`.img`文件。由于本系统最终是运行在软盘这一古老的介质上，所以不需要为虚拟机分配多少资源。大概分配1GB的ROM和512MB的RAM就可以了。然后进入`设置`->`存储`->`Floppy`中加入`.img`文件即可。

`.bat`可以认为是`Windows`上存储命令的文本文件，但其还有更强大的功能，类似于`Linux`中的`Shell`。这里仅使用最基础的功能。

在笔者提供的基于`NASM`改编的汇编程序下我们可以写成第一个汇编语言程序作为第一个内核，我称之为`LC_Kernel_0.0.1`。主要完成了从软盘进行系统加载的功能，中间的一些输入和输出现在还不知道是什么意思，之后会明白。

```assembly
; Kernel version 0.0.1
; TAB = 4

; 这段代码跑在标准FAT32格式软盘专用代码

DB    0xeb, 0x4e, 0x90    
DB    "LCKERNEL"          ; 启动区名称（必须8 Bytes）
DW    512                 ; 扇区大小
DB    1                   ; 簇（cluster）的大小
DW    1                   ; FAT起始位置，从1扇区开始
DB    2                   ; FAT个数，必须为2
DW    224                 ; 根目录大小，一般设成224
DW    2880                ; 磁盘大小，一般为2880扇区
DB    0xf0                ; 磁盘种类，固定值 1111 0000
DW    9                   ; FAT长度，必须为9扇区
DW    18                  ; 一个磁道有多少扇区，必须为18
DW    2                   ; 磁头数，必须为2
DD    0                   ; 不使用分区，必须为0
DD    2880                ; 重写一次磁盘大小（？）
DB    0, 0, 0x29          ; 固定写法
DD    0xffffffff          ; 卷标号码（？）
DB    "Lichen_Kernel"     ; 磁盘名称（13 Bytes）
DB    "FAT12   "         ; 磁盘格式名称（必须8 Bytes）
RESB  18                  ; 先空出 18 Bytes

; main
    DB    0xb8, 0x00, 0x00, 0x8e, 0xd0, 0xbc, 0x00, 0x7c
    DB    0x8e, 0xd8, 0x8e, 0xc0, 0xbe, 0x74, 0x7c, 0x8a
    DB    0x04, 0x83, 0xc6, 0x01, 0x3c, 0x00, 0x74, 0x09
    DB    0xb4, 0x0e, 0xbb, 0x0f, 0x00, 0xcd, 0x10, 0xeb
    DB    0xee, 0xf4, 0xeb, 0xfd

; Show Slogan
    DB    0x0a, 0x0a      ; 换行
    DB    "Lichen_Kernel 0.0.1"     ; Slogan
    DB    0x0a
    DB    0               ; （？）

    RESB  0x1fe-$         ; 填写0x00到地址0x001fe
    DB    0x55, 0xaa      ; 结束

; 启动区以外的输出
DB    0xf0, 0xff, 0xff, 0x00, 0x00, 0x00, 0x00, 0x00
RESB  4600
DB    0xf0, 0xff, 0xff, 0x00, 0x00, 0x00, 0x00, 0x00
RESB  1469432
```

这里对这段代码做一个小小的解释。

此处`$`的含义是这一行现在的字节数。

`FAT32`是一个可以运行在`Windows`和`UNIX`系统上的文件格式。最大可存`4GB`的文件，采用3级索引的方式。

软盘的第一个扇区是启动区`boot selector`，系统从第一个扇区开始读盘，软盘中，一个软盘`512 Bytes`。软盘大小为`1440KB`，即`1440 * 1024 = 1474560 Bytes`。则有`1440 * 2 = 2880`个扇区。在第一扇区，若最后两字节不以`0x55 0xAA`结束，则计算机将不认为软盘上有需启动的程序，就不能加载系统程序。

`IPL`是启动程序装载器。一般把`IPL`写入启动区，通过`IPL`加载在主存其他地方的系统本身，这样就可以让系统大小突破第一扇区512 Bytes的限制。

### 2. 进化到汇编语言

上一个版本的`Kernel`最大的问题出在程序主体`main`部分。这里是用二进制数来表示指令，这样的工作在吃过午饭过后就只有上帝能够看懂了。汇编语言就是将指令映射成人能看懂的部分，然后通过汇编器编译`compile`成中间产物，然后再由连接器将中间产物和`.dll`连接后形成可执行二进制文件。这样可以使可读性提高，同时兼顾执行效率。

然后将上一个`Kernel`用汇编语言改写（`NASM`语法）改写。

```assembly
; Kernel version 0.0.2
; FAT32

ORG 0x7c00

; 记录用于标准FAT12格式的软盘

JMP   entry
DB    0X90
DB    "LCKERNEL"          ; 启动区名称（必须8 Bytes）
DW    512                 ; 扇区大小
DB    1                   ; 簇（cluster）的大小
DW    1                   ; FAT起始位置，从1扇区开始
DB    2                   ; FAT个数，必须为2
DW    224                 ; 根目录大小，一般设成224
DW    2880                ; 磁盘大小，一般为2880扇区
DB    0xf0                ; 磁盘种类，固定值 1111 0000
DW    9                   ; FAT长度，必须为9扇区
DW    18                  ; 一个磁道有多少扇区，必须为18
DW    2                   ; 磁头数，必须为2
DD    0                   ; 不使用分区，必须为0
DD    2880                ; 重写一次磁盘大小（？）
DB    0, 0, 0x29          ; 固定写法
DD    0xffffffff          ; 卷标号码（？）
DB    "Lichen_Kernel"     ; 磁盘名称（13 Bytes）
DB    "FAT12   "          ; 磁盘格式名称（必须8 Bytes）
RESB  18                  ; 先空出 18 Bytes

; main
entry:
    MOV        AX, 0          ; 初始化寄存器
    MOV        SS, AX
    MOV        SP, 0X7c00
    MOV        DS, AX
    MOV        ES, AX

    MOV        SI, msg

putloop:
    MOV        AL, [SI]
    ADD        SI, 1           ; 给SI加1
    CMP        AL, 0

    JE         fin
    MOV        AH, 0X0e        ; 显示一个文字
    MOV        BX, 15          ; 指定字符颜色
    INT        0X10            ; 调用显卡BIOS
    JMP        putloop

fin:
    HLT                        ; 令CPU停止，等待命令
    JMP        fin             ; 无限循环

msg:
    DB         0x0a, 0x0a      ; 换行两次
    DB         "Lichen Kernel 0.0.2"
    DB         0x0a            ; 换行
    DB         0               ; 结束
    RESB  	   0x7dfe-$         ; 填写0x00到地址0x001fe
    DB    	   0x55, 0xaa      ; 结束

; 启动区以外的输出
DB    0xf0, 0xff, 0xff, 0x00, 0x00, 0x00, 0x00, 0x00
RESB  4600
DB    0xf0, 0xff, 0xff, 0x00, 0x00, 0x00, 0x00, 0x00
RESB  1469432
```

`ORG`将机器指令装载在何处，使用该命令后`$`的含义表示读入的内存地址。

`JMP`，跳转指令。

x86体系结构中的寄存器有如下几个。

| 代号  |        名称         |
| :---: | :-----------------: |
| `AX`  |     accumulator     |
| `BX`  |        base         |
| `CX`  |       counter       |
| `DX`  |        data         |
| `SP`  |    stack pointer    |
| `BP`  |    base pointer     |
| `SI`  |    source index     |
| `DI`  |  destination index  |
| `ES`  |    extra segment    |
| `CS`  |    code segment     |
| `SS`  |    stack segment    |
| `DS`  |    data segment     |
| `PSW` | program status word |
| `FS`  |        null         |

其中，`AX, BX, CX, DX, SP, BP, SI, DI`这8个寄存器可以拓展到32位，这8个中的前四个可以分成两个8位寄存器。

方括号表示内为内存地址。内存使用小端存储。

数据类型有`BYTE, WORD, DWORD` 。可以指定内存地址处存的是什么类型的数据。

`HLT`，让`CPU`停止工作进入待机状态，当输入设备对其输出时，`CPU`被唤醒。

至于为何要将程序放在`0x7c:0x00`。这是因为主存分配所致，有些主存实现被写好，如`BIOS`。如图为主存分配图。

在这种系统中，软盘可视为外存。

![8086内存分配图](https://pic.imgdb.cn/item/60d44b28844ef46bb2109302.png)

这张图片也记录了CPU主存的使用结构。

![](https://pic.imgdb.cn/item/60d44b3e844ef46bb211037a.png)

如下图所示，记录了BIOS数据区的具体内容。

![BIOS](https://pic.imgdb.cn/item/60d44b57844ef46bb2118669.png)


`Makefile`是一种简单的批处理文件。

> \# 注释
>
> file0 : file1 file 2 file 3
>
> ​	command0
>
> ​	command1
>
> ​	...
>
> file4: ...
>
> ​	...

语句含义为若存在`file1, file2, file3`，就执行下面的命令，否则不执行。可以用`\`续行，以便于命令过长时可以换行。可以制作一个`MakeFile`文件将程序中向软盘其余部分写入的操作从Kernel文件中删除，这个操作主要用到了`edimg.exe`。这个文件的原理是读入一个空白磁盘镜像文件，将`IPL`部分写入启动区。

首先，将写入磁盘部分的操作删除，形成`Kernel 0.0.3`，此时的`Kernel`只有在第一扇区的`IPL`部分。

```assembly
; Kernel version 0.0.3
; IPL Only
; FAT32

ORG 0x7c00

; 记录用于标准FAT12格式的软盘

JMP   entry
DB    0X90
DB    "LCKERNEL"          ; 启动区名称（必须8 Bytes）
DW    512                 ; 扇区大小
DB    1                   ; 簇（cluster）的大小
DW    1                   ; FAT起始位置，从1扇区开始
DB    2                   ; FAT个数，必须为2
DW    224                 ; 根目录大小，一般设成224
DW    2880                ; 磁盘大小，一般为2880扇区
DB    0xf0                ; 磁盘种类，固定值 1111 0000
DW    9                   ; FAT长度，必须为9扇区
DW    18                  ; 一个磁道有多少扇区，必须为18
DW    2                   ; 磁头数，必须为2
DD    0                   ; 不使用分区，必须为0
DD    2880                ; 重写一次磁盘大小（？）
DB    0, 0, 0x29          ; 固定写法
DD    0xffffffff          ; 卷标号码（？）
DB    "Lichen_Kernel"     ; 磁盘名称（13 Bytes）
DB    "FAT12   "          ; 磁盘格式名称（必须8 Bytes）
RESB  18                  ; 先空出 18 Bytes

; main
entry:
    MOV        AX, 0          ; 初始化寄存器
    MOV        SS, AX
    MOV        SP, 0X7c00
    MOV        DS, AX
    MOV        ES, AX

    MOV        SI, msg

putloop:
    MOV        AL, [SI]
    ADD        SI, 1           ; 给SI加1
    CMP        AL, 0
    JE         fin
    MOV        AH, 0X0e        ; 显示一个文字
    MOV        BX, 15          ; 指定字符颜色
    INT        0X10            ; 调用显卡BIOS
    JMP        putloop

fin:
    HLT                        ; 令CPU停止，等待命令
    JMP        fin             ; 无限循环

msg:
    DB         0x0a, 0x0a      ; 换行两次
    DB         "Lichen_Kernel 0.0.3"
    DB         0x0a            ; 换行
    DB         0               ; 结束
    RESB  	   0x7dfe-$         ; 填写0x00到地址0x001fe
    DB    	   0x55, 0xaa      ; 结束
```

然后编写`Makefile`文件来指明`IPL`部分的编译方法，通过`make.bat`，`install.bat`实现编译、安装等工作。

```makefile
# How to make files.

Kernel0_0_3.bin: Kernel0_0_3.nas Makefile
	nask Kernel0_0_3.nas Kernel0_0_3.bin Kernel0_0_3.lst
Kernel0_0_3.img: Kernel0_0_3.bin Makefile
# Format: edimg imgin: [file] wbinimg src: [.bin file] len: [bytes] from: [num] to: [num] imgout: [.img file]
# \ equals to \n  
	edimg imgin:../sources/fdimg0at.tek \
	wbinimg src:Kernel0_0_3.bin len:512 from:0 to:0 imgout:Kernel0_0_3.img
```

可以再添加一些命令使得`make`文件更加智能

```makefile
img:
	make -r Kernel0_0_3.img
asm:
	make -r Kernel0_0_3.bin
```

然后，只需要执行`make img`便能生成`Kernel0_0_3.img`文件了。

当前文件不需要根目录`/`，这是导致刚开始报错的原因。

### 3. 制作`IPL`并进化到C语言

前三个`Kernel`严格来讲就是在`IPL`所在的第一扇区写程序，而不是真正的装载程序，比较像`汇编语言`这门课的粗略入门，下面将正式写一个可以装载程序的真正的`IPL`。并且导入的程序是用一种更高级的`C Language`来写的。

新写成的`Kernel0_0_4`如下，同上一个版本相比，增加了读盘的动作。

```assembly
; Kernel version 0.0.4
; Real IPL
; FAT32

ORG 0x7c00

; 记录用于标准FAT12格式的软盘

JMP   entry
DB    0X90
DB    "LCKERNEL"          ; 启动区名称（必须8 Bytes）
DW    512                 ; 扇区大小
DB    1                   ; 簇（cluster）的大小
DW    1                   ; FAT起始位置，从1扇区开始
DB    2                   ; FAT个数，必须为2
DW    224                 ; 根目录大小，一般设成224
DW    2880                ; 磁盘大小，一般为2880扇区
DB    0xf0                ; 磁盘种类，固定值 1111 0000
DW    9                   ; FAT长度，必须为9扇区
DW    18                  ; 一个磁道有多少扇区，必须为18
DW    2                   ; 磁头数，必须为2
DD    0                   ; 不使用分区，必须为0
DD    2880                ; 重写一次磁盘大小（？）
DB    0, 0, 0x29          ; 固定写法
DD    0xffffffff          ; 卷标号码（？）
DB    "Lichen_Kernel"     ; 磁盘名称（13 Bytes）
DB    "FAT12   "          ; 磁盘格式名称（必须8 Bytes）
RESB  18                  ; 先空出 18 Bytes

; main
entry:
    MOV        AX, 0     
    MOV        SS, AX            ; 设置栈段为0
    MOV        SP, 0x7c00        ; 设置栈顶指针为7c00
    MOV        DS, AX            ; 设置数据段为0，防止读写时指向错误的地方

    MOV        AX, 0x0820        ; 读盘相关的设置，将缓存段设置为0x0820
    MOV        ES, AX            ; 缓存地址为ES:BX，最大缓存大约为1MB
    MOV        CH, 0             ; 0柱面
    MOV        DH, 0             ; 0磁头
    MOV        CL, 2             ; 2扇区
    
    MOV        AH, 0x02
    MOV        AL, 1
    MOV        BX, 0
    MOV        DL, 0x00          ; 驱动器号
    INT        0x13              ; 调用BIOS读盘中断
    JC         error             ; 当PSW的进位标志为1时跳转

error:
    MOV        SI, msg

fin:
    HLT
    JMP        fin

putloop:
    MOV        AL, [SI]          ; 实际上的地址是 DS:SI 
    ADD        SI, 1
    CMP        AL, 0
    JE         fin               ; 当受到0的结束标志时，跳转到fin
    MOV        AH, 0x0e
    MOV        BX, 15
    INT        0x10
    JMP        putloop

msg:
    DB         0x0a, 0x0a
    DB         "Error Occured"
    DB         0x0a
    DB         0
    RESB       0x7dfe-$          ; 保证写入长度为0x1fe=510 Bytes

    DB         0x55, 0xaa        ; 结束标志
```

解释几个新东西，第一个是中断，可以理解成`BIOS`提供给我们的接口供我们`invoke`（调用）。这个程序运用到了`0x10`中断和`0x13`中断。前者是用来控制显卡在`Terminal`下进行显示使用的，后者是`Disk I/O, Verify, Seek`中断，简单来说负责磁盘读写、检验、寻道。这个程序用`0磁头`读了`柱面0`的`2扇区`。具体的BIOS中断参数有相关的文档可以参考，这里可以开个新坑，把这个文档翻译了，~~考完以后~~。

这里学习到了`Makefile`文件的新写法，即变量的配置。通过变量的设置可以使命令变短，更方便执行。这里我猜测变量的机制应该就是`C Language`的`define`中的字符替换。定义变量的方法如下。

> \# Var_Name = [string]
>
> \# $(Var_Name) [params]
>
> \# equals to [string] [params]

这里将`Makefile`文件也列出来。

```makefile
# Makefile for Kernel0_0_4

# How to define a variable in Makefile.
EMPTYIMG = ../sources/fdimg0at.tek
MAKE = make -r

Kernel0_0_4.bin : Kernel0_0_4.nas Makefile
	nask Kernel0_0_4.nas Kernel0_0_4.bin Kernel0_0_4.lst

Kernel0_0_4.img: Kernel0_0_4.bin Makefile
	edimg imgin:$(EMPTYIMG) wbinimg src:Kernel0_0_4.bin len:512 from:0 to:0 imgout:Kernel0_0_4.img

img:
	$(MAKE) Kernel0_0_4.img
```

软盘运行起来不可靠，可以设置一个重试机制。重试超过5次就转到`error`部分，读盘发生错误，但没超过5次，就调用重置驱动器的中断，然后继续跳转到retry段读盘。再次改进后一次读取18个扇区。

```assembly
; main
entry:
    MOV        AX, 0     
    MOV        SS, AX            ; 设置栈段为0
    MOV        SP, 0x7c00        ; 设置栈顶指针为7c00
    MOV        DS, AX            ; 设置数据段为0，防止读写时指向错误的地方

    MOV        AX, 0x0820        ; 读盘相关的设置，将缓存段设置为0x0820
    MOV        ES, AX            ; 缓存地址为ES:BX，最大缓存大约为1MB
    MOV        CH, 0             ; 0柱面
    MOV        DH, 0             ; 0磁头
    MOV        CL, 2             ; 2扇区

readloop:
    MOV        SI, 0             ; 记录失败次数的寄存器

retry:    
    MOV        AH, 0x02          ; 读盘
    MOV        AL, 1
    MOV        BX, 0
    MOV        DL, 0x00          ; 驱动器号0
    INT        0x13              ; 调用BIOS读盘中断
    JNC        next               ; 跳转到next部分
    ADD        SI, 1
    CMP        SI, 5
    JAE        error             ; 若AE>5就跳转到error
    MOV        AH, 0x00
    MOV        DL, 0x00
    INT        0x13              ; 重置驱动器
    JMP        retry             ; 循环

next:
    MOV        AX, ES            ; 以下三步将地址往后加了0x0200
    ADD        AX, 0x0020
    MOV        ES, AX
    ADD        CL, 1
    CMP        CL, 18            
    JBE        readloop          ; 若CL<18则转移到readloop
error:
    MOV        SI, msg

fin:
    HLT
    JMP        fin

putloop:
    MOV        AL, [SI]          ; 实际上的地址是 DS:SI 
    ADD        SI, 1
    CMP        AL, 0
    JE         fin               ; 当受到0的结束标志时，跳转到fin
    MOV        AH, 0x0e
    MOV        BX, 15
    INT        0x10
    JMP        putloop

msg:
    DB         0x0a, 0x0a
    DB         "Error Occured"
    DB         0x0a
    DB         0
    RESB       0x7dfe-$          ; 保证写入长度为0x1fe=510 Bytes

    DB         0x55, 0xaa        ; 结束标志
```

这里出现的新指令`JAE`是`Jump if above or equal`。当大于等于时跳转。`JBE`是`Jump if below or equal`。当小于等于跳转。

再次改造，读取10个柱面。

```assembly
; main
entry:
    MOV        AX, 0     
    MOV        SS, AX            ; 设置栈段为0
    MOV        SP, 0x7c00        ; 设置栈顶指针为7c00
    MOV        DS, AX            ; 设置数据段为0，防止读写时指向错误的地方

    MOV        AX, 0x0820        ; 读盘相关的设置，将缓存段设置为0x0820
    MOV        ES, AX            ; 缓存地址为ES:BX，最大缓存大约为1MB
    MOV        CH, 0             ; 0柱面
    MOV        DH, 0             ; 0磁头
    MOV        CL, 2             ; 2扇区

readloop:
    MOV        SI, 0             ; 记录失败次数的寄存器

retry:    
    MOV        AH, 0x02          ; 读盘
    MOV        AL, 1
    MOV        BX, 0
    MOV        DL, 0x00          ; 驱动器号0
    INT        0x13              ; 调用BIOS读盘中断
    JNC        next               ; 跳转到next部分
    ADD        SI, 1
    CMP        SI, 5
    JAE        error             ; 若AE>5就跳转到error
    MOV        AH, 0x00
    MOV        DL, 0x00
    INT        0x13              ; 重置驱动器
    JMP        retry             ; 循环

next:
    MOV        AX, ES            ; 以下三步将地址往后加了0x0200
    ADD        AX, 0x0020
    MOV        ES, AX
    ADD        CL, 1
    CMP        CL, 18            
    JBE        readloop          ; 若CL<18则转移到readloop
    MOV        CL, 1
    ADD        DH, 1
    CMP        DH, 2
    JB         readloop
    MOV        DH, 0
    ADD        CH, 1
    CMP        CH, CYLS
    JB         readloop

error:
    MOV        SI, msg

fin:
    HLT
    JMP        fin
```

我们可以观察到，我们系统的执行部分实际上为`fin`部分。让CPU停止工作并循环执行`fin`。我们可以通过上述对程序的改进察觉到操作系统启动的过程。

> 1. 操作系统从介质地址0处开始读`IPL`，它位于外存柱面0的第一扇区。
> 2. `IPL`中的装载指令装入主存的0x7c00 : 0x000处。CPU的指令指针便能从主存中读到指令并通过微程序执行。
> 3. 这些指令的目的是将外存其他部分的程序读到主存中并通过CPU执行。

现在，我们将操作系统可以理解成两大部分：`IPL`和`操作系统主体`。

我们这时将该程序分成两部分，启动区命令为`IPL.nas`，主体命名为`main.nas`。

我们尝试将一段最简单的代码写入软盘，代码如下：

```assembly
fin:
	HLT
	JMP        fin
```

将其编译为`.sys`文件，然后写入`.img`。这里改写以下`Makefile`文件。不是特别理解`edimg`命令的意思，~~这个软件没有help~~。

```makefile
# Makefile for Kernel0_0_4

# How to define a variable in Makefile.
EMPTYIMG = ../sources/fdimg0at.tek
MAKE = make -r

Kernel0_0_4.bin: Kernel0_0_4.nas Makefile
	nask Kernel0_0_4.nas Kernel0_0_4.bin Kernel0_0_4.lst

main.sys: main.nas Kernel0_0_4.bin Makefile
	nask main.nas main.sys main.lst
main.img: main.sys Kernel0_0_4.bin Makefile
	edimg imgin:$(EMPTYIMG) \
	wbinimg src:Kernel0_0_4.bin len:512 from:0 to:0 \
	copy from:main.sys to:@: \
	imgout:main.img

img:
	$(MAKE) main.img
```

下面用二进制编辑器分析程序`main.nas`被装入的位置。如下图可知，文件名的起始存储地址为`0x002600`，文件内容存储起始地址`0x004200`。

![](https://pic.imgdb.cn/item/60d4492b844ef46bb20645db.png)

![](https://pic.imgdb.cn/item/60d449ea844ef46bb20a100c.png)

而磁盘外存内容装载到内存的起始地址是`0x08000`，所以磁盘`0x004200`处的内容应该位于`0x8000+0x4200=0x0c200`处。我们下面改写程序。将main.sys变得复杂一些。

```assembly
; Lichen_kernel main
; version 0.0.1

ORG        0xc200

MOV        AL, 0x13        ; VGA显卡，320*200*8位颜色
MOV        AH, 0x00
INT        0x10

fin:
    HLT
    JMP    fin
```

这里要对Kernel0_0_4.nas进行改造，改造如下

```assembly
; Kernel version 0.0.4
; Real IPL
; FAT32

ORG 0x7c00
CYLS EQU 10

; 记录用于标准FAT12格式的软盘

JMP   entry
DB    0X90
DB    "LCKERNEL"          ; 启动区名称（必须8 Bytes）
DW    512                 ; 扇区大小
DB    1                   ; 簇（cluster）的大小
DW    1                   ; FAT起始位置，从1扇区开始
DB    2                   ; FAT个数，必须为2
DW    224                 ; 根目录大小，一般设成224
DW    2880                ; 磁盘大小，一般为2880扇区
DB    0xf0                ; 磁盘种类，固定值 1111 0000
DW    9                   ; FAT长度，必须为9扇区
DW    18                  ; 一个磁道有多少扇区，必须为18
DW    2                   ; 磁头数，必须为2
DD    0                   ; 不使用分区，必须为0
DD    2880                ; 重写一次磁盘大小（？）
DB    0, 0, 0x29          ; 固定写法
DD    0xffffffff          ; 卷标号码（？）
DB    "Lichen_Kernel"     ; 磁盘名称（13 Bytes）
DB    "FAT12   "          ; 磁盘格式名称（必须8 Bytes）
RESB  18                  ; 先空出 18 Bytes

; main
entry:
    MOV        AX, 0     
    MOV        SS, AX            ; 设置栈段为0
    MOV        SP, 0x7c00        ; 设置栈顶指针为7c00
    MOV        DS, AX            ; 设置数据段为0，防止读写时指向错误的地方

    MOV        AX, 0x0820        ; 读盘相关的设置，将缓存段设置为0x0820
    MOV        ES, AX            ; 缓存地址为ES:BX，最大缓存大约为1MB
    MOV        CH, 0             ; 0柱面
    MOV        DH, 0             ; 0磁头
    MOV        CL, 2             ; 2扇区

readloop:
    MOV        SI, 0             ; 记录失败次数的寄存器

retry:    
    MOV        AH, 0x02          ; 读盘
    MOV        AL, 1
    MOV        BX, 0
    MOV        DL, 0x00          ; 驱动器号0
    INT        0x13              ; 调用BIOS读盘中断
    JNC        next               ; 跳转到next部分
    ADD        SI, 1
    CMP        SI, 5
    JAE        error             ; 若AE>5就跳转到error
    MOV        AH, 0x00
    MOV        DL, 0x00
    INT        0x13              ; 重置驱动器
    JMP        retry             ; 循环

next:
    MOV        AX, ES            ; 以下三步将地址往后加了0x0200
    ADD        AX, 0x0020
    MOV        ES, AX
    ADD        CL, 1
    CMP        CL, 18            
    JBE        readloop          ; 若CL<18则转移到readloop
    MOV        CL, 1
    ADD        DH, 1
    CMP        DH, 2
    JB         readloop
    MOV        DH, 0
    ADD        CH, 1
    CMP        CH, CYLS
    JB         readloop

    MOV        [0x0ff0], CH      ; IPL记下读取到的地方
    JMP        0xc200            ; 跳转到main.sys执行

error:
    MOV        SI, msg

  fin:
      HLT
      JMP        fin

putloop:
    MOV        AL, [SI]          ; 实际上的地址是 DS:SI 
    ADD        SI, 1
    CMP        AL, 0
    JE         fin               ; 当受到0的结束标志时，跳转到fin
    MOV        AH, 0x0e
    MOV        BX, 15
    INT        0x10
    JMP        putloop

msg:
    DB         0x0a, 0x0a
    DB         "Error Occured"
    DB         0x0a
    DB         0
    RESB       0x7dfe-$          ; 保证写入长度为0x1fe=510 Bytes

    DB         0x55, 0xaa        ; 结束标志
```

可以看到读盘完成后有一个通过`0x0ff0`地址存储当前读盘位置，并跳转到`main.sys`所在外存位置并运行。这里打开了一个黑色的画布，与以往不同的是这次没有了闪烁的光标（可以理解为被遮住了）。接下来导入`C语言`。

由于文件结构越来越复杂，从此刻开始，版本号只在文件夹显示，文件将被命名为`IPL.nas`和`main.nas`。

此刻，我们将进入`32位`进行开发，因为32位拥有更大的寻址能力，CPU的自我保护功能。同样，你将放弃调用`BIOS`的权利。

所以，我们要在进入32位之前将所有的`BIOS`设置好。

```assembly
; Lichen_kernel main
; version 0.0.2

CYLS	EQU		0x0ff0			; 设定启动区
LEDS	EQU		0x0ff1
VMODE	EQU		0x0ff2			; 设定颜色位数
SCRNX	EQU		0x0ff4			; 分辨率x
SCRNY	EQU		0x0ff6			; 分辨率y
VRAM	EQU		0x0ff8			; 图像缓冲区开始地址

ORG        0xc200

MOV        AL, 0x13        ; VGA显卡，320*200*8位颜色
MOV        AH, 0x00
INT        0x10

MOV        BYTE [VMODE], 8;
MOV        WORD [SCRNX], 320;
MOV        WORD [SCRNY], 200;
MOV        DWORD [VRAM], 0x000a0000

; 用BIOS取得键盘指示灯状态
MOV        AH, 0x02
INT        0x16
MOV        [LEDS], AL

fin:
    HLT
    JMP    fin
```

处理完所有`BIOS`相关内容后就可以引入C语言程序了。由于引入C语言要进行额外的操作，所以对`main.nas`进行了一些修改，现在可能还不知道这些修改的目的，不过接下来会知道的（这段代码是我copy的）。

C语言程序`bootpack.c`的源代码如下。

```c
// Lichen_kernel 0.0.4
// fist C program in order to trap into a loop.

void HariMain(void) {
    fin:
        goto fin;
}
```

我们都知道，C语言程序必须编译成机器语言后才能被执行。在此系统中，如果要将C程序编译成机器语言，需要经过以下几步。

> 1. 用`gcc`把C程序编译成`gas`汇编语言程序
> 2. 将gas汇编语言程序编译成nask汇编语言程序
> 3. 将`.nas`编译成`.obj`
> 4. 将`.obj`链接生成`.bim`
> 5. 从`.bim`文件生成`.hrb`机器语言程序

这里解释一下。`.bim`中的bim是`binary image`的缩写，即二进制映像文件。

同样的，此时我们也需要对这个`MakeFile`文件进行修改。

```makefile
# Makefile for 0.0.4
# This is the first version including C file.

# How to define a variable in Makefile.
EMPTYIMG = ../sources/fdimg0at.tek
MAKE = make -r
INCPATH = ../sources/haribote
RULEFILE = ../sources/haribote/haribote.rul

# generate IPL.bin & main.bin & naskfunc.obj
IPL.bin: IPL.nas Makefile
	nask IPL.nas IPL.bin IPL.lst

main.bin: main.nas Makefile
	nask main.nas main.bin main.lst

naskfunc.obj : naskfunc.nas Makefile
	nask naskfunc.nas naskfunc.obj naskfunc.lst

# compile and link bootpack.c
bootpack.gas: bootpack.c Makefile
	cc1 -I $(INCPATH) -Os -Wall -quiet \
	-o bootpack.gas bootpack.C
bootpack.nas: bootpack.gas Makefile
	gas2nask -a bootpack.gas bootpack.nas
bootpack.obj: bootpack.nas Makefile
	nask bootpack.nas bootpack.obj bootpack.lst
bootpack.bim: bootpack.obj naskfunc.obj Makefile
	obj2bim @$(RULEFILE) out:bootpack.bim stack:3136k map:bootpack.map \
	bootpack.obj naskfunc.obj
bootpack.hrb: bootpack.bim Makefile
	bim2hrb bootpack.bim bootpack.hrb 0

# generate main.sys & final img
main.sys: main.bin bootpack.hrb Makefile
	copy /B main.bin+bootpack.hrb main.sys
final.img: main.sys IPL.bin Makefile
	edimg imgin:$(EMPTYIMG) \
	wbinimg src:IPL.bin len:512 from:0 to:0 \
	copy from:main.sys to:@: \
	imgout:final.img

img:
	$(MAKE) final.img
```

执行后可以生成`final.img`。这个效果和之前的效果没什么不同QAQ。

这里如果报错可以查看以下两个文件，第一个是`.rul`文件，需要将路径改为你的实际路径，第二次检查可以检查以下你的C语言程序的主函数名字是否正确。

这样第三部分就完结了，感觉这时候已经写了很多东西，但重复的东西很多，其实内容不是很多。

坚持要写操作系统的话需要仔细研究研究它这个`link`的步骤，研究研究为什么`bootpack.c`的主函数命名必须为`HariMain`，看看改掉哪里就可以将主函数名字改掉。

*暂时没有找到可以改掉这个主函数名字的方法*。

**要抽时间去翻译一下8086 BIOS 中断的含义**

### 4. 实现内存写入与特定画面显示

