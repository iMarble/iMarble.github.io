---
title: Program To Process and Assembly Basics - I
date: 2025-05-06
categories: [Binary Exploitation, Pwning]
tags: [binary, exploitation, pwning, process, assembly]
author: 0
---

# Process
In the previous post, we talked briefly about binaries and processes. Now it's time to go a bit deeper into the processes since we'll be dealing with them all the time.  
What happens when a binary is loaded into memory?  
It becomes a process... But what else? A few other things also happen in memory that should be taken into consideration when we are diving into binary exploitation.  

## Process Structure
Once the program is loaded, it becomes a process, and it gets its own memory space (which is isolated from other processes). The OS assigns memory regions to the process, including:

- **Text Segment**: The executable code.
- **Data Segment**: Global and static variables.
- **Heap**: Dynamic memory, allocated at runtime (like when you use malloc() in C or new in C++).
- **Stack**: Function calls and local variables.
- **Program Counter (PC)**: Points to the current instruction that the CPU will execute next.

### Text Segment
The text segment (our code) — is it in English, Urdu, or Latin? No. The text segment is made of machine instructions. But luckily, we can use debuggers (e.g., GDB) to help us understand these machine instructions in a language called assembly language. This is where assembly language comes into play and helps us comprehend what the actual code is doing (by scratching our heads) without having the actual high-level code.

Debuggers can also help us understand what's going on in other parts like:
- What's in the Data Segment, Heap, and Stack
- State of Flags (We'll talk about this later)
- Registers
- And much more

# Assembly Language
According to [Wikipedia](https://en.wikipedia.org/wiki/Assembly_language):

```
In computer programming, assembly language, often referred to simply as assembly and commonly abbreviated as ASM or asm, is any low-level programming language with a very strong correspondence between the instructions in the language and the architecture's machine code instructions.
```


We don't have to write whole programs in assembly language, and we can't. We just have to read and understand the instructions. That's it — that's all. The more efficiently we can understand what an instruction is doing, the better decisions we can make and the more weaknesses we can find.

We'll cover the following points briefly regarding assembly language:
- Registers
- Flags
- Basic Instructions
- Jumps

There are also two flavors (writing styles) for assembly language:
- Intel
- AT&T

We'll focus on Intel flavor; it's easier and less messy.

## Registers
The processor includes some internal memory storage locations, called registers.  
Registers act as temporary holding areas for data, instructions, or memory addresses, enabling the CPU to process information quickly and effectively.

### Types of Registers
1. General Purpose Registers (GPRs)
2. Special Purpose Registers
3. Flags Register

### General Purpose Registers
These are the registers that don't have any special meaning and are just used to store data temporarily. These are the following:
- RAX, EAX, AX, AH, AL
- RBX, EBX, BX, BH, BL
- RCX, ECX, CX, CH, CL
- RDX, EDX, DX, DH, DL
- RSI, ESI, SI
- RDI, EDI, DI
- RBP, EBP, BP
- RSP, ESP, SP
- R8–R15, R8D–R15D, R8W–R15W, R8B–R15B (64-bit registers and their 32, 16, and 8-bit versions)

The naming convention is as follows:  
- **R** = 64-bit  
- **E** = 32-bit  
- **AX** = 16-bit  
- **AH** = upper 8 bits  
- **AL** = lower 8 bits  

### Special Purpose Registers
Some of the GPRs also serve special purposes. These are:
- **RIP** (Instruction Pointer): Points to the current instruction in the program (64-bit).
- **RFLAGS**: Stores flags that control the CPU’s operations and reflect the outcomes of operations.
- **CS** (Code Segment): Points to the segment containing the executable code.
- **SS** (Stack Segment): Points to the segment containing the stack.
- **DS** (Data Segment): Points to the segment containing data variables.
- **ES**, **FS**, **GS**: Additional segment registers used for specific purposes (such as handling data for thread-local storage, etc.).

We will talk about remaining Assembly Basics in Part-II.