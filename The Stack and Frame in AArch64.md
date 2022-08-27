# The Stack and Frame in AArch64

## When should use them?

When you find that your program is getting bigger and bigger, you should find some ways to make your code more readable. The first method in your mind is possibly using function. You will find that in 64-bits arm world, it's more easier to use function than in x86.

For example, if you have a main program called `main.s` and a function called `foo` in `foo.s`. You can invoke the function very easily by this instruction:

```assembly
// something done before
bl foo
// something will do after return
```

The secret behind the easy use of invoking function is that there is not just one register `PC` that can record the return address. In AArch64, the processor has at least two specific registers about the address of instruction, `pc` and `lr`. `pc` register records the address of next instruction that waits to be executed. `lr` register records the return address. So there is no need for programmers to put `pc` and parameters to stack as in x86.

However, things will change if you consider some situations that you call function which calls another function. In these cases, you must store the `lr` and `sp` into somewhere you can visit later. That is because `lr` can only record the latest return address. For example, if you want to call function `foo1` which calls `foo2`, you can see that `lr` records the return address of main function after calling `foo1`. When `foo1` calls `foo2`, `lr` will update the value to the return address of `foo1`. If you don't store the return address of main function, you will lose that. Here is a brief graph that demonstrates well.

```
main ---> foo1
          foo1 ---> foo2
          foo1 <--- foo2
main <-x- foo1
```

## Why use two registers for handling stack?

That's because of the alignment feature of ARM processors and clumsy mechanism of stack in AArch64. ARM requires all its instructions and data be aligned in order to visit them more conveniently. `sp` register must be aligned with 16 bytes. Stack and frame grow by decreasing the address. Therefore, the element of stack has a size of 16 bytes. `sp` records the top of stack, and the last unit of `sp` must be zero.

But 16 bytes means too large for many situations. an element of stack can store two values of general registers, which are 64 bits. If you have some small size values, for example, 32 bits or 16 bits, that's clumsy to store them into a 16 bytes element respectively. That's why you need `fp` register. `fp` doesn't have a limit that must be aligned with 16 bytes. So you can use `fp` register more flexible and decrease the waste of stack.

## Some good methods to use stack wisely

* Clean your stack after using;
* Use macros to simple your command;
* Store any usable information to your stack or frame;