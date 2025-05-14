---
title: Buffer Overflow Basics
date: 2025-05-14
categories: [Binary Exploitation, Pwning]
tags: [binary, exploitation, pwning, process, assembly]
author: 0
---

# Buffer Overflow
A buffer overflow occurs when a program writes more data to a buffer (like an array or a string) than it's supposed to hold. This extra data can overwrite nearby memory, even the return address—which is where the magic (or the vulnerability) happens.
Examples:
Declaring an array of 5 values and accidentally taking 6 values from user.
Declaring a string of length 10 but the user can input more than 10.

## Why Buffer Overflows Matter in Pwning
In assembly, and in low-level languages like C, there are `no bounds checks` unless the programmer adds them `manually`. This means if we can control the input, we might:

- Overwrite variables
- Overwrite function return addresses
- Hijack the program flow
- Redirect execution to shellcode or any location we want

This is exactly what attackers exploit.

## Simple Example in C
It is easy to understand with an example, so here's a simple example in C
```
void vulnerable() {
    char buffer[16];
    gets(buffer);  // Dangerous! No size check
}
```

## What’s After the Buffer?
As we discussed earlier in stack and stack frame post, When a function is called, a stack frame is created. Inside that frame:

- The local variables (like your `buffer`) are stored first (at lower memory addresses).
- Then the `saved base pointer` (ebp) is pushed.
- And then the `return address` (which tells the program where to go back after the function ends).

Here’s a visual layout if your buffer is 16 bytes

```
|-------------------------|  ← Lower memory (top of stack)
|      buffer[16]         |  ← Local variable
|-------------------------|
|      saved EBP          |  ← Base pointer before function call
|-------------------------|
|    return address       |  ← Execution jumps here on `ret`
|-------------------------|  ← Higher memory (bottom of frame)
```

If your input exceeds 16 bytes, you will:

- First `overwrite saved EBP`, which is not critical but can crash stack traces.
- Then `overwrite the return address`, which is very critical.

If we can control what goes into the return address, we control where the program jumps after the function ends. That’s the `core idea` of a stack-based buffer overflow.

## Example Payload
Here's an example payload for buffer overflow.
If we want to redirect execution to 0x08048444, we can send:
```
python3 -c 'print("A"*16 + "B"*4 + "\x44\x84\x04\x08")' | ./vuln_binary
```

In this we are sending
1. 16 A's to overflow the buffer
2. 4 B's to overwrite the saved EBP
3. return address of our choice in `little endian` format

This was all from this post. In the next posts, we'll cover
- How to find the exact offset to control the return address
- How to create a working exploit
- stack canaries / protections like NX and how to bypass them!