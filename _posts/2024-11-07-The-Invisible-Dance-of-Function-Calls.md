---
layout: post
---

## Table of Contents
1. [Introduction](#introduction)
2. [Runtime Memory Fundamentals](#runtime-memory-fundamentals)
3. [Function Call Lifecycle](#function-call-lifecycle)
4. [Further Reading](#further-reading)
5. [Conclusion](#conclusion)

## Introduction

Modern software development is built upon the fundamental concept of functions (also called procedures) - units of code that perform specific tasks. The prevailing focus is on quickly writing reliable and portable code. Concerns about memory efficiency and CPU performance seem like outdated relic from the past. But have you ever think about what actually happens when you call a function? How are local variables managed without interfering between them? The answer lies in one of the most elegant mechanisms in computer science: the stack frame.

## Runtime Memory Fundamentals

The stack is a fundamental linear data structure that follows a LIFO (Last In First Out) model. For example, imagine that plates are added to the top of a the stack in a cafeteria. When someone needs a plate, they take one from the top. If you want to access a plate lower, you must first remove all the plates on top of it, one by one, until reach the requested plate. Similarly, in a stack structure the most recent data (or function call) added is the first to be removed.

In the context of CPU's runtime environment, the stack is used primarily for function calls, local variable storage, and managing the execution context. When a function is called, a new stack frame is created to store data such as the function's parameters and the return address.

CPU stack is managed and affected by several registers:

- SP (Stack Pointer): Points to the top of the stack.
- BP (Base Pointer): Points to the bottom of the stack.
- IP (Instruction Pointer): Points to the next instruction to be executed by the CPU.

## Function Call Lifecycle

Consider this C program:

```c
#include <stdio.h>

int sum(int a, int b)
{
  return a + b;
}

void main()
{
  int result = sum(10, 20);
}
```

In this example, we have two functions, `main` and `sum`. In order to understand a little bit more about CPU stack, let's debug the C program and analyze the Assembly code after compilation.

For the `main` function:

```c
0x000055555555513d <+0>:	push   %rbp
0x000055555555513e <+1>:	mov    %rsp,%rbp
0x0000555555555141 <+4>:	sub    $0x10,%rsp
0x0000555555555145 <+8>:	mov    $0x14,%esi
0x000055555555514a <+13>:	mov    $0xa,%edi
0x000055555555514f <+18>:	call   0x555555555129 <sum>
0x0000555555555154 <+23>:	mov    %eax,-0x4(%rbp)
0x0000555555555157 <+26>:	nop
0x0000555555555158 <+27>:	leave
0x0000555555555159 <+28>:	ret
```

Suppose you are in the `main` function. The first thing the program does is save the value of the `rbp` register (which points to the base of the caller's stack frame) onto the stack. This is helpful because allow the `main` function to restore the stack to its correct state when it finishes.

Then, the `rbp` register is set to the current value of the `rsp` stack pointer to establish the stack frame for the `main` function. Finally, there are local variables (e.g `result`), so space for them is reserved onto the stack with the `sub` instruction.

The previous process is the so-called **Function Prologue**.

The next step is set up the arguments required for the `sum` function. In this case, the `edi` register stores the first argument (`a` for `sum`) and the `esi` register stores the second argument (`b` for `sum`). After, the `sum` function is called, it pushes the return address onto the stack and then jumps to the address of the `sum` function to start execution. 

This is the Assembly code for the `sum` function:

```c
0x0000555555555129 <+0>:	push   %rbp
0x000055555555512a <+1>:	mov    %rsp,%rbp
0x000055555555512d <+4>:	mov    %edi,-0x4(%rbp)
0x0000555555555130 <+7>:	mov    %esi,-0x8(%rbp)
0x0000555555555133 <+10>:	mov    -0x4(%rbp),%edx
0x0000555555555136 <+13>:	mov    -0x8(%rbp),%eax
0x0000555555555139 <+16>:	add    %edx,%eax
0x000055555555513b <+18>:	pop    %rbp
0x000055555555513c <+19>:	ret
```

It follows the same process as the `main` function. The function starts saving the value of the base pointer from the calling function (`main` in this case) and setting up the stack frame, next, it stores the arguments into local variables onto the stack (through `edi` and `esi` registers).

In the next instruction the function moves the arguments into registers (`edx`, `eax`), performs the operation, and stores the result in `eax`.

After, the function needs to clean up before returning. It restores the previous function's base pointer from `main` and returns to the address that was saved on the stack after the `call` instruction.

Finally, the control returns to `main`. The instruction `mov %eax, -0x4(%rbp)` takes the return value of the sum stored in `%eax` register and stores it in the `result` local variable. When `main` finishes, the clean up instructions are executed and the control is returned to the operating system. This is the so-called **Function Epilogue**.

## Further Reading

For those interested in diving deeper into the stack and low-lever programming, here are some valuable resources:

1. **Books**
   - "Introduction to 64 bit Intel Assembly Language Programming for Linux" by Ray Seyfarth
   - "Computer Organization and Design" by David A. Patterson and John L. Hennesy
   - "The Art of 64-bit Assembly" by Randall Hyde

2. **Online Resources**
   - GCC Internals Documentation: https://gcc.gnu.org/onlinedocs/gccint/
   - LLVM User Guides: https://llvm.org/docs/UserGuides.html
   - x86 Assembly Guide: https://www.cs.virginia.edu/~evans/cs216/guides/x86.html

## Conclusion

The stack is not just a fundamental part of the system, it is the backbone of function call management, memory allocation, recursion handling and crucial security mechanisms. Mastering its knowledge is essential for writing highly optimized, performant code.

Moreover, the importance of the stack for the runtime behavior makes it a critical consideration for the security of software systems. Vulnerabilities such as buffer overflows exploit the stack working, highlighting the necessity of understand this concept.

