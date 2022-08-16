---
title: 一个 macOS 汇编的分析（M1 芯片）
date: 2021-09-28
tag: macOS
categories: 汇编语言
top: false
mathjax: false
---

## 一个 macOS 汇编的分析（M1 芯片）

### 1. 代码

源文件 `main.c` 代码如下

```c
#include <stdio.h>

void swap(long long int v[], size_t k) {
    __asm__ __volatile__ (
                          "lsl x10, x1, #3\n"
                          "add x10, x0, x10\n"
                          "ldur x9, [x10, #0]\n"
                          "ldur x11, [x10, #8]\n"
                          "stur x11, [x10, #0]\n"
                          "stur x9, [x10, #8]");
}

int main(int argc, const char * argv[]) {
    // insert code here...
    long long int num[3] = {1, 2, 3};
    swap(num, 1);
    printf("%lld %lld\n", num[1], num[2]);
    return 0;
}
```

swap 函数实现了语句 `v[k]  <--> v[k+1]`，将两个元素进行交换，注意，这里使用了内联汇编。

执行以后的 `main.s` 文件代码如下

```assembly
	.section	__TEXT,__text,regular,pure_instructions
	.build_version macos, 11, 0	sdk_version 11, 3
	.globl	_swap                           ; -- Begin function swap
	.p2align	2
_swap:                                  ; @swap
	.cfi_startproc
; %bb.0:
	sub	sp, sp, #16                     ; =16
	.cfi_def_cfa_offset 16
	str	x0, [sp, #8]
	str	x1, [sp]
	; InlineAsm Start
	lsl	x10, x1, #3
	add	x10, x0, x10
	ldur	x9, [x10]
	ldur	x11, [x10, #8]
	stur	x11, [x10]
	stur	x9, [x10, #8]
	; InlineAsm End
	add	sp, sp, #16                     ; =16
	ret
	.cfi_endproc
                                        ; -- End function
	.globl	_main                           ; -- Begin function main
	.p2align	2
_main:                                  ; @main
	.cfi_startproc
; %bb.0:
	sub	sp, sp, #80                     ; =80
	stp	x29, x30, [sp, #64]             ; 16-byte Folded Spill
	add	x29, sp, #64                    ; =64
	.cfi_def_cfa w29, 16
	.cfi_offset w30, -8
	.cfi_offset w29, -16
	adrp	x8, ___stack_chk_guard@GOTPAGE
	ldr	x8, [x8, ___stack_chk_guard@GOTPAGEOFF]
	ldr	x8, [x8]
	stur	x8, [x29, #-8]
	str	wzr, [sp, #28]
	str	w0, [sp, #24]
	str	x1, [sp, #16]
	adrp	x8, l___const.main.num@PAGE
	add	x8, x8, l___const.main.num@PAGEOFF
	ldr	q0, [x8]
	add	x0, sp, #32                     ; =32
	str	q0, [sp, #32]
	ldr	x8, [x8, #16]
	str	x8, [sp, #48]
	mov	x1, #1
	bl	_swap
	ldr	x8, [sp, #40]
	ldr	x9, [sp, #48]
	adrp	x0, l_.str@PAGE
	add	x0, x0, l_.str@PAGEOFF
	mov	x10, sp
	str	x8, [x10]
	str	x9, [x10, #8]
	bl	_printf
	adrp	x8, ___stack_chk_guard@GOTPAGE
	ldr	x8, [x8, ___stack_chk_guard@GOTPAGEOFF]
	ldr	x8, [x8]
	ldur	x9, [x29, #-8]
	subs	x8, x8, x9
	b.ne	LBB1_2
; %bb.1:
	mov	w8, #0
	mov	x0, x8
	ldp	x29, x30, [sp, #64]             ; 16-byte Folded Reload
	add	sp, sp, #80                     ; =80
	ret
LBB1_2:
	bl	___stack_chk_fail
	.cfi_endproc
                                        ; -- End function
	.section	__TEXT,__const
	.p2align	3                               ; @__const.main.num
l___const.main.num:
	.quad	1                               ; 0x1
	.quad	2                               ; 0x2
	.quad	3                               ; 0x3

	.section	__TEXT,__cstring,cstring_literals
l_.str:                                 ; @.str
	.asciz	"%lld %lld\007n"

.subsections_via_symbols
```

### 2. 分析

```assembly
	.section	__TEXT,__text,regular,pure_instructions
	.build_version macos, 11, 0	sdk_version 11, 3
```

首先，定义了一个节（section）属于__Text 段的 _text 节，属于常规（regular）类型，属性为纯指令（pure_instructions）。

随后指定了构建系统版本（macOS 11.0）和 SDK 版本（11.3）。

然后开始定义 swap 函数。

```assembly
	.globl	_swap                           ; -- Begin function swap
	.p2align	2
_swap:                                  ; @swap
	.cfi_startproc
; %bb.0:
	sub	sp, sp, #16                     ; =16
	.cfi_def_cfa_offset 16
	str	x0, [sp, #8]
	str	x1, [sp]
	; InlineAsm Start
	lsl	x10, x1, #3
	add	x10, x0, x10
	ldur	x9, [x10]
	ldur	x11, [x10, #8]
	stur	x11, [x10]
	stur	x9, [x10, #8]
	; InlineAsm End
	add	sp, sp, #16                     ; =16
	ret
	.cfi_endproc
```

首先前三行申明了函数名称和指令对齐方式，其中`.p2align `伪指令指定指令以 2^x 字节对齐，由于 arm 指令集定长为 32 位，即 4 字节，所以对齐方式为 4 字节对齐。`.cfi_startproc`和`.cfi_endproc`定义了一段代码的开始与结束，在这之间写汇编指令。

这里要说明的是，arm64 要求栈的对齐方式为16字节，由于 swap 的这两个参数均为 64 位，即 8 字节，所以只需要开一个 16 字节的栈就可以了。和 x86 类似，栈也是从高地址向低地址生长，且参数由右向左入栈，所以 x0 存放的是 v 数组的基地址，x1 存放的是 k 的值，由于 `long long int` 数据类型一个占 8 字节，所以需要将 k * 8 才能找到 `&v[k]`。然后通过一系列 `load/store` 交换内存这两个元素的值，最后清空栈，也就是将 sp 加 16，然后通过 ret 指令设置 pc 程序计数寄存器返回到调用者。

然后开始定义 main 函数。在操作系统看来，`main` 函数和其他函数并没有什么不同，由于它是用户编写程序的入口，调用者是操作系统，执行完毕或出现异常之后要返回到操作系统，所以 main 函数第一步需要做的是定义对栈，然后将操作系统在执行 main 函数的 fp 信息和 sp 信息存储下来。

```assembly
	sub	sp, sp, #80                     ; =80
	stp	x29, x30, [sp, #64]             ; 16-byte Folded Spill
	add	x29, sp, #64                    ; =64
	.cfi_def_cfa w29, 16
	.cfi_offset w30, -8
	.cfi_offset w29, -16
```

首先是建立堆栈，这里的对栈也是 16 字节对齐。这里建立了一个五个元素的对栈。其中，第一个元素有 16 字节，可以存放 fp 和 lr 的值。然后栈顶指向第四个元素的下面第一个字节表示现在堆栈为空。`.cfi_def_cfa` 其中 `cfi` 指的是 call from info，是 DWARF 2.0 规定的堆栈信息。`cfa` 指的是 `canonical frame address` 规范堆栈框，这是定义了该程序所有函数的基地址，其基地址是原先 x29 寄存器加 16，即我们开的80字节堆栈的栈底。然后又定义了 fp 和 sp 相对于 cfa 的偏移。整个对栈此时的情况如下。

| 相对地址（以执行程序前 sp 地址为相对零地址） | 存放内容 | 字节      |
| -------------------------------------------- | -------- | --------- |
| 0                                            |          |           |
| +16                                          |          |           |
| +32                                          |          |           |
| +48                                          |          |           |
| +64                                          | FP       | 低 8 字节 |
|                                              | LR       | 高 8 字节 |
| +80                                          | CFA      |           |

然后通过暂存寄存器 x8 去检查堆栈是否发生错误，指令如下。

```assembly
	adrp	x8, ___stack_chk_guard@GOTPAGE
	ldr	x8, [x8, ___stack_chk_guard@GOTPAGEOFF]
	ldr	x8, [x8]
```

首先，先从编译器堆栈保护函数处获取页号将基地址对应到 x8 上，然后加载页偏移处 x8 的值，这也是一个地址，最后通过这个地址找到该值并且将其送入到 x8 中。接着，这几步看不大懂。

```assembly
	stur	x8, [x29, #-8]
	str	wzr, [sp, #28]
	str	w0, [sp, #24]
	str	x1, [sp, #16]
```

然后在堆栈中存储程序中的数据

```assembly
	adrp	x8, l___const.main.num@PAGE
	add	x8, x8, l___const.main.num@PAGEOFF
	ldr	q0, [x8]
	add	x0, sp, #32                     ; =32
	str	q0, [sp, #32]
	ldr	x8, [x8, #16]
	str	x8, [sp, #48]
	mov	x1, #1
```

前两句赋给 x8 寄存器 &num，然后将从 sp + 32 开始的 4 个 `long long int` 大小的地方存 num 数组。注意，这里的存储是小端存储。

之后就是执行 `printf` 函数了。

### 3. 从 .s 编译到可执行程序
.s --> .o

```
as -o <filename>.o <filename>.s
```

.o --> executive file (binary file)

```
ld -o <filename> <filename>.o
-lSysten
-syslibroot `xcrun -sdk macosx --show-sdk-path`
-e start
-arch arm64
```

* -lSystem: tell the linker to link executable with `libSystem.dylib`
* -syslibroot: it's mandatory to tell the linker where to find `libSystem.dylib`
* `xcrun -sdk -macosx --show-sdk-path`: dynamically use the currently active version of Xcode
* -e start: tell the linker `start` is the entrypoint of our program (Darwin expects `main` by default)
* -arch arm64: tell the linker our architecture is arm64, which is optional if you use apple silicon mac