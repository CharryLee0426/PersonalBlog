# Some Data about Speed of Multiplication and Addition

## 1. Running 34,359,738,367 (0x7ffffffff) times

| program | real time | user  | system | instruction retired | cycles elapsed |
| ------- | --------- | ----- | ------ | ------------------- | -------------- |
| mul     | 10.80     | 10.77 | 0.03   | 206,174,586,404     | 34,460,405,906 |
| add     | 10.81     | 10.77 | 0.02   | 206,174,961,103     | 34,452,388,733 |
| add     | 10.80     | 10.76 | 0.03   | 206,174,196,589     | 34,463,690,145 |
| mul     | 10.81     | 10.77 | 0.03   | 206,174,331,889     | 34,466,289,391 |

## 2. Running 137,438,953,471 (0x1fffffffff) times

| program | real time | user  | system | instruction retired | cycles elapsed  |
| ------- | --------- | ----- | ------ | ------------------- | --------------- |
| add     | 43.12     | 42.98 | 0.12   | 824,684,047,224     | 137,845,540,756 |
| mul     | 43.26     | 43.05 | 0.19   | 824,706,262,533     | 138,088,583,489 |
| mul     | 43.11     | 42.99 | 0.09   | 824,678,165,222     | 137,763,496,059 |
| add     | 43.11     | 42.99 | 0.10   | 824,680,759,525     | 137,775,698,560 |



## 3. Code

add.s

```assembly
.include "debug.s"
.global main
.p2align 2

main:
        mov     w1,     #1
        mov     w2,     #2
        mov     w4,     w1
        mov     w5,     w2
        mov     x3,     #0xffff
        movk    x3,     #0xffff,    lsl #16
        movk    x3,     #0x0007,    lsl #32
        movk    x3,     #0x0000,    lsl #48
loop:   subs    x3,     x3,     #1
        b.eq    print
        add     w1,     w1,     w2
        mov     w1,     w4
        mov     w2,     w5
        b       loop
print:  printReg    1

				// return 0
        mov     x0,     #0
        ret
```

mul.s

```assembly
.include "debug.s"
.global main
.p2align 2

main:
        mov     w1,     #1
        mov     w2,     #2
        mov     w4,     w1
        mov     w5,     w2
        mov     x3,     #0xffff
        movk    x3,     #0xffff,    lsl #16
        movk    x3,     #0x0007,    lsl #32
        movk    x3,     #0x0000,    lsl #48
loop:   subs    x3,     x3,     #1
        b.eq    print
        smull   x1,     w1,     w2
        mov     w1,     w4
        mov     w2,     w5
        b       loop
print:  printReg    1

				// return 0
        mov     x0,     #0
        ret
```

**Same points in these two programs**

* Same registers
  * w1, w2: operation register;
  * x1: destination register;
  * x3: index register;
  * x0: kernel parameter;
* Same statements
  * w1 = 1, w2 = 2;
  * w4 = w1, w5 = w2;
  * same loop logic;
  * w1 and w2 are assigned by their initial value;
  * same printReg:
    * store x0-x18 and lr (general registers) to stack;
    * put the parameters from x0 to x3;
    * put x1-x3 to stack;
    * invoke `_printf`;
    * load x0-x18 and lr (general registers) from stack;
  * return 0;

The only difference between these two programs is add and smull.

## 4. First Conclusion

These two given tables illustrate the performance difference between multiplication and addition instructions for integers arithmetic. Overall, it proves that addition instructions execute faster tha multiplication instructions for integers.

Firstly take a look at the first table. This table sorted those records by time and these two programs compiled just once. Time usage of these executions are very near. The total time is about 10.80s and system time is 0.03s approximately. However, `add` can be completed by 10.76s at least, which is 0.01s faster than `mul` can. As for instruction retired, `mul` has 206,174,586,404 retired instructions, which is above 400,000 less than `add` has. If they execute again, they both can get about 400,000 retired instructions decrease. They both have similar cycle elapsed in their two executions. `mul` needs more cycles than `add` in order to complete its task.

Regarding the second tables, time, instruction retired and cycle elapsed all have a very huge growth because the loop round number is much larger. The total time in four recordings is about 43s and system time is 0.10s approximately. In first round(i.e. the first two records), `add` needs 42.98s to run, which is 0.07s less than `mul` needs. With the optimisation of system, they can have the same performance, 42.99s. They need 840 billions retired instruction and about 137 cycles to finish the task.

All in all, with the improvement made by processor, `mul` and `add` can have a similar performance. But at first time, `add` performs better without any optimisation. Thus we can get the conclusion that addition instructions execute faster than multiplications.

## 5. Code Improve

I learn that C programming language provides efficient library to measure the clock and time used in the program. Therefore I update the code in order to get rid of the time command. All in all, using in-code method to measure the clock and time used is more accuracy and elegant than using time command provided by UNIX. Here are the two updated codes.

add.s

```assembly
.include "debug.s"
.global main
.p2align 2

main:
        mov     w1,     #1
        mov     w2,     #2
        mov     w4,     w1
        mov     w5,     w2
        mov     x3,     #0xffff
        movk    x3,     #0xffff,    lsl #16
        movk    x3,     #0x0007,    lsl #32
        movk    x3,     #0x0000,    lsl #48

        // start clock
        stp     fp,     lr,     [sp, #-16]!
        bl      _clock
        ldp     fp,     lr,     [sp],   #16
        mov     x8,     x0
        str     x8,     [sp, #-16]!

        // do add for x3-1 times
loop:   subs    x3,     x3,     #1
        b.eq    after
        add     w1,     w1,     w2
        mov     w1,     w4
        mov     w2,     w5
        b       loop

        // end clock
after:  stp     fp,     lr,     [sp, #-16]!
        bl      _clock
        ldp     fp,     lr,     [sp],   #16
        mov     x9,     x0
        str     x9,     [sp, #8]

        // calculate
        mov     x10,    #0x4240
        movk    x10,    #0xf,   lsl #16
        ldp     x8,     x9,     [sp]
        ldr     x7,     [sp]
        scvtf   d8,     x8
        scvtf   d9,     x9
        scvtf   d10,    x10
        fsub    d8,     d9,     d8
        fdiv    d8,     d8,     d10
        fmov    x8,     d8

        // results
        printStr        "Add instructions speed test"
        printStr        "Start Clock:"
        printReg        7
        printStr        "End Clock:"
        printReg        9
        printStr        "Time used:"
        printDouble     8
        printStr        ""
        

		// return 0
        mov     x0,     #0
        ret
```

mul.s

```assembly
.include "debug.s"
.global main
.p2align 2

main:
        mov     w1,     #1
        mov     w2,     #2
        mov     w4,     w1
        mov     w5,     w2
        mov     x3,     #0xffff
        movk    x3,     #0xffff,    lsl #16
        movk    x3,     #0x0007,    lsl #32
        movk    x3,     #0x0000,    lsl #48

        // start clock
        stp     fp,     lr,     [sp, #-16]!
        bl      _clock
        ldp     fp,     lr,     [sp],   #16
        mov     x8,     x0
        str     x8,     [sp, #-16]!

        // do add for x3-1 times
loop:   subs    x3,     x3,     #1
        b.eq    after
        smull   x1,     w1,     w2
        mov     w1,     w4
        mov     w2,     w5
        b       loop

        // end clock
after:  stp     fp,     lr,     [sp, #-16]!
        bl      _clock
        ldp     fp,     lr,     [sp],   #16
        mov     x9,     x0
        str     x9,     [sp, #8]

        // calculate
        mov     x10,    #0x4240
        movk    x10,    #0xf,   lsl #16
        ldp     x8,     x9,     [sp]
        ldr     x7,     [sp]
        scvtf   d8,     x8
        scvtf   d9,     x9
        scvtf   d10,    x10
        fsub    d8,     d9,     d8
        fdiv    d8,     d8,     d10
        fmov    x8,     d8

        // results
        printStr        "Mul instructions speed test"
        printStr        "Start Clock:"
        printReg        7
        printStr        "End Clock:"
        printReg        9
        printStr        "Time used:"
        printDouble     8
        printStr        ""
        

		// return 0
        mov     x0,     #0
        ret
```

