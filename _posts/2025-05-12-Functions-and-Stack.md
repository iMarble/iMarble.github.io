---
title: Functions and Stack
date: 2025-05-12
categories: [Binary Exploitation, Pwning]
tags: [binary, exploitation, pwning, process, assembly]
author: 0
---

In today's post, we'll first talk about functions in assembly language. Later, we'll explore how functions work and what calling conventions are.

# Procedures/Functions
Functions in assembly language are just like functions in any other programming language. They are a set of instructions that are executed when the function is called.
Example:
Consider you want to add two numbers and then subtract a specific number from the result. The steps are the same every time; only the values change.
So, instead of rewriting those instructions again and again, you can write a function to do this, and just call it with different values.

These input values are called function arguments, and when we provide them during a function call, we refer to them as parameters passed to the function.

## Defining a Function
Functions can be defined using `labels` in assembly language.
A `label` is simply a keyword placed at the start of a line. The assembler uses it to identify the location of that block of code.
Example:
```
Function1:
    add eax, ebx
    sub eax, 1
```
Here we defined a function called Function1.

### Parameters in Assembly Functions
There's no manual way to directly set function parameters like in high-level languages.
Instead, how parameters are passed is decided by the calling convention used by the system or compiler.

Some common calling conventions are:
- cdecl
- stdcall
- fastcall
- sysv_amd64 (for 64-bit Linux)

We'll look into each of these with examples.

## Calling Conventions
Calling conventions define how functions receive parameters, how the return value is passed, and who is responsible for cleaning up the stack after a function call.

Different operating systems and compilers can use different conventions. Here are the most common ones used in pwning and reverse engineering:

### cdecl (C Declaration) — Mostly used in 32-bit Linux
- Arguments are passed right to left on the stack
- Return value is in eax
- Caller cleans up the stack (meaning the one who calls the function is responsible for adjusting the stack after the function call)
Example:
```
push 3
push 2
call add_and_subtract
add esp, 8     ; caller cleans 2 arguments (2 * 4 bytes)
```

### stdcall — Mostly used in 32-bit Windows
- Same as cdecl, but the callee cleans up the stack (the function itself does it)
- Return value is in eax
Example:
```
push 3
push 2
call add_and_subtract
; No need to clean stack — callee will do it
```

### fastcall — Used in some Windows environments
- First two arguments passed in ecx and edx
- Remaining arguments go on the stack
- Return value is in eax
- Cleanup depends on the specific implementation
Example:
```
mov ecx, 2
mov edx, 3
call add_and_subtract
```

### sysv_amd64 — Used in 64-bit Linux (very important for pwning)
- First six arguments are passed in registers: `rdi, rsi, rdx, rcx, r8, r9`
- Additional arguments go on the stack
- Return value is in rax
- Caller is responsible for stack cleanup
Example:
```
mov rdi, 2      ; arg1
mov rsi, 3      ; arg2
call add_and_subtract
```

# Stack
The stack is a data structure that works in a LIFO (Last In First Out) manner.
This means values are stacked on top of each other, and when retrieving, you get the last value first, and the first value last.

## Stack in Assembly
In assembly language, the stack plays a very important role. It is involved in:

- Memory management
- Function calls and returns
- Passing arguments
- Saving return addresses and register states

The stack is growing `downward in memory`. This means when new data is pushed onto the stack, the stack pointer (esp in 32-bit or rsp in 64-bit) `decreases`.

## Basic Stack Instructions
### Push
Pushes a value onto the stack.
This decreases the stack pointer and stores the value at the new location.

Example:
```
push eax
; esp = esp - 4
; [esp] = value of eax
```

### Pop
Pops the top value from the stack into a register.
This reads the value from the top of the stack and then increases the stack pointer.

Example:
```
pop ebx
; ebx = [esp]
; esp = esp + 4
```

## Stack Behavior Example
```
push 1        ; stack: [1]
push 2        ; stack: [2, 1]
pop eax       ; eax = 2, stack: [1]
pop ebx       ; ebx = 1
```

# Stack Frames

When a function is called, a stack frame is created. This frame is a portion of the stack that contains:

- Function parameters
- Local variables
- The return address (where to go back after the function finishes)
- Saved values of registers (like ebp)

This layout helps manage the function’s execution in a clean, isolated way.

## Key Registers
- `esp (Stack Pointer)`: Always points to the top of the current stack.
- `ebp (Base Pointer)`: Marks the base of the current function's stack frame.

## How a Stack Frame is Built

When a function is called, the following usually happens:
### 1. Call instruction
Pushes the return address onto the stack (i.e., where to return after function completes).

### 2. Function prologue
This is the setup phase inside the function:
```
push ebp        ; Save old base pointer
mov ebp, esp    ; Set up new base pointer
sub esp, XX     ; Allocate space for local variables
```

### 3. Function body

Use stack space (via offsets from ebp) to access parameters and locals.

### 4. Function epilogue
Clean up and return:
```
mov esp, ebp    ; Restore stack pointer
pop ebp         ; Restore old base pointer
ret             ; Return to caller using saved return address
```

## Visual Stack Frame Example (32-bit)

Imagine calling this C function:
```
int func(int a, int b) {
    int c = a + b;
    return c;
}
```

When func is called, the stack might look like this:

[esp+0x0]   ← return address
[esp+0x4]   ← argument b
[esp+0x8]   ← argument a
[ebp-0x4]   ← local variable c

And ebp will point to [esp+0x0] after mov ebp, esp.

## Accessing Data in Stack Frame

    Function arguments: accessed using positive offsets from ebp
    (e.g., [ebp + 8], [ebp + 12])

    Local variables: accessed using negative offsets from ebp
    (e.g., [ebp - 4], [ebp - 8])

From next posts we'll start buffer overflows and see how stack plays a role in it!