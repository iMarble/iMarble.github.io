---
title: Assembly Basics - II
date: 2025-05-06
categories: [Binary Exploitation, Pwning]
tags: [binary, exploitation, pwning, process, assembly]
author: 0
---

In previous post we talked about processes, how and why assembly is required. We also covered some of the basics of the assembly language. In today's post we will extend the assembly knowledge and read about
- Flags
- Instructions
- Jumps
- Functions

# Flags
Flags in assembly are bits which can be 0 or 1 depending upon different coditions. These are stored in a register i.e EFLAGS register. Every bit in this indicate a flag. If the flag is set then it is 1, if it is cleared then it is 0.
```
EFLAGS register in x86 processors is a 32-bit register that holds status flags that indicate the result of computations and control the CPU's operations
```
These flags are set or cleared by instructions and used for
- comparisons
- conditional jumps
- loops

These flags indicate the outcome of arithmetic, logical, and other operations, and are used to control the flow of execution.

## Important Flags
There are many different flags, but the ones use mostly in pwning are
- ZF (Zero Flag): Set in different scenarios, mostly when the comparison is equal then it is set, otherwise cleared.
- CF (Carry Flag): Set when there's a carry out or borrow in arithmetic operations; often used to detect unsigned overflows.
- SF (Sign Flag): Reflects the sign of the result; set if the result is negative.
- OF (Overflow Flag): Set when a signed overflow occurs in arithmetic operations (e.g., adding two positive numbers gives a negative result).

# Instructions
Assembly language consists of a number of instructions, but the ones we will mostly encounter in pwning are the following:

# Referrencing conventions
The following are the conventions for referencing in Intel flavor:
- [] square brackets represent memory value wherever used. E.g., [eax] means the memory value at the address stored in eax.
- eax, ebx, ecx are registers.
- 1, 2, and 3 are immediate values
- 0x10, 0xA, 0xBC are immediate values in hexadecimal (mostly used).

## mov instruction
Moves data from source to destination.
The source can be a register, memory, or an immediate value.
The destination can only be a register or memory.
Examples:
```
mov dest, source
mov eax, 1
mov eax, ebx
mov [eax], ecx
```

## and instruction
Performs a bitwise AND between two operands.
Used to clear bits (e.g., zeroing out lower bytes).
Examples:
```
and eax, 0x0F      ; keeps only the lower 4 bits of eax
and byte [esp+4], 0xF0
```

## xor instruction
Performs a bitwise XOR between two operands.
Commonly used to zero out a register because xor reg, reg is shorter and faster than mov reg, 0.
Examples:
```
xor eax, eax       ; sets eax to 0
xor ebx, ebx       ; sets ebx to 0
```

## sub instruction
Subtracts the source from the destination and stores the result in the destination.
Often used for adjusting stack pointers or counters.
Examples:
```
sub esp, 0x20      ; allocate 32 bytes on the stack
sub eax, ebx
```

## add instruction
Adds the source to the destination and stores the result in the destination.
Frequently used alongside sub for stack or loop adjustments.
Examples:
```
add esp, 0x10      ; clean up stack space
add eax, 4
```

## cmp instruction
Compares two operands by internally doing a subtraction (but does not store the result).
Used before conditional jumps like je, jne, etc.
Examples:
```
cmp eax, 0         ; compare eax with 0
cmp eax, ebx
```

These were some of the normal instructions which we will encounter. We shall cover instructions related to `stack`, `jumps` and `function calls` also.

# Jumps
Jumps in assembly language are used to control the flow of the program. Just like in high-level languages, we have conditionals, in assembly we use jumps. Jumps can be `conditional` as well as `unconditional`. 

## Unconditional Jumps
Unconditional jumps are simple jumps used to move from one instruction to another.
For example, consider that we are on instruction number 1 and we want to skip instructions 2, 3, and 4. We can use a simple jump to go directly to instruction number 5.
The major difference to note between `jmp` and `call` is:
- `call` saves the return address (so the flow can return after the function finishes)
- `jmp` does not return; it simply continues from the destination.

### Example
```
0x1     statement1
0x2     statement2
0x3     ...
0x4     ...
0x5     jmp 0x9
0x6     ... (skipped since we jumped in 0x5)
0x7     ...
0x8     ...
0x9     flow continues from here
```

## Conditional Jumps
Conditional jumps are jumps taken based on certain conditions.
They are mostly used immediately after a `cmp` instruction. They rely on CPU flags `(ZF, SF, OF, CF)` to decide whether to jump.

Here are some commonly used conditional jumps and their respective conditions:

## Signed Comparisons
These jumps are used when working with signed numbers.

### JG (Jump if Greater)
This jump is taken when one `signed number` is greater than other `signed number`. This will set the ZF if the number is greater
Condition: ZF == 0 and SF == OF
Explanation: No overflow and not equal = strictly greater.

Example:
```
cmp -1, -2
JG 0x10 (this jump will be taken since -1 is greater than -2)
```

### JGE (Jump if Greater or Equal)
This jump is taken if one signed number is greater than or equal to the other.
Condition: SF == OF
Explanation: No signed overflow, result is valid (greater). Basically cmp instruction is just subtraction so if the number is less than the other number then SF flag would be SET.
Example:
```
cmp 5, 2
jg 0x10 (this will be taken)

cmp 2, 1
jg 0x124 (this will not be taken)
```

### JL (Jump if Less)
This jump is taken if one signed number is less than the other.
Condition: SF != OF
Explanation: Signed overflow indicates the result is logically negative.
Example:
```
cmp eax, ebx
jl label  ; jump if eax < ebx (signed)
```

### JLE (Jump if Less or Equal)
Taken if one signed number is less than or equal to the other.
Condition: ZF == 1 or SF != OF
Explanation: Either equal, or signed result is less.

Example:
```
cmp eax, ebx
jle label  ; jump if eax <= ebx (signed)
```

## Unsigned Comparisons
These jumps are used when working with unsigned numbers (positive-only values, like memory addresses).
### JA (Jump if Above)
Taken if the first unsigned number is strictly greater than the other.
Condition: CF == 0 and ZF == 0

Example:
```
cmp eax, ebx
ja label  ; jump if eax > ebx (unsigned)
```

### JAE or JNB (Jump if Above or Equal / Not Below)

Taken if the first unsigned number is greater than or equal to the other number.
Condition: CF == 0

Example:
```
cmp eax, ebx
jae label   ; jump if eax >= ebx (unsigned)
jnb label   ; same as above
```

### JB (Jump if Below)
Taken if the first unsigned number is strictly less.
Condition: CF == 1

Example:
```
cmp eax, ebx
jb label  ; jump if eax < ebx (unsigned)
```

### JBE or JNA (Jump if Below or Equal / Not Above)

Taken if the first unsigned number is less than or equal.
Condition: CF == 1 or ZF == 1

Example:
```
cmp eax, ebx
jbe label   ; jump if eax <= ebx (unsigned)
jna label   ; same as above
```

## Common to Both
These jumps are same for both signed and unsiged numbers.

### JE (Jump if Equal)
Taken if the values are equal.
Condition: ZF == 1

Example:
```
cmp eax, ebx
je label   ; jump if eax == ebx
```

### JNE (Jump if Not Equal)

Taken if the values are not equal.
Condition: ZF == 0

Example:
```
cmp eax, ebx
jne label  ; jump if eax != ebx
```

That was all from this post. In the next post we will see about STACK and Function calling in assembly.