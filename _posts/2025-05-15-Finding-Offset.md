---
title: Offsets in Buffer Overflow
date: 2025-05-15
categories: [Binary Exploitation, Pwning]
tags: [binary, exploitation, pwning, process, assembly]
author: 0
---

# Finding the Exact Offset
When performing a buffer overflow, you need to know exactly how many bytes it takes to reach the return address. Instead of guessing, we use a special pattern that helps us locate the overflow point precisely.

# Steps
First of all we should have [pwntools](https://github.com/Gallopsled/pwntools) installed, for making our life easier.
Then we can follow these steps:

## 1. Generate a Unique Pattern
Use `cyclic` from `pwntools`. It creates a long string where each 4-byte chunk is unique, so you can easily identify the exact overwrite point.
Example
```
from pwn import *
pattern = cyclic(100)
print(pattern)
```
Or directly in terminal:
```
cyclic 100
```

## 2. Run the Vulnerable Program
Send the pattern as input to the vulnerable binary:
```
./vuln_binary <<< $(cyclic 100)
```
This will cause the program to crash and overwrite the return address with some unique part of the pattern.

## 3. Inspect in GDB
Now launch the program inside GDB and trigger the crash again:
```
gdb ./vuln_binary
(gdb) run <<< $(cyclic 100)
```

After it crashes, check the value of the return address (usually in the instruction pointer eip on 32-bit):
```
(gdb) info registers
```

Let’s say we see:
```
eip = 0x6161616c
```

## 4. Find the Offset
Now use cyclic_find to figure out how many bytes into the pattern that value appears:
```
cyclic -l 0x6161616c
```
Or in Python:
```
from pwn import *
cyclic_find(0x6161616c)
```

Output might be: 44, 45, 46, ..., n
So now we know:

    It takes exactly n bytes to reach the return address.

## Payload
Now that we know the offset, we can build our payload like this:
```
payload = b"A" * 44 + b"BBBB"  # 'BBBB' will overwrite return address
```
we can replace BBBB with any 4-byte address we want to redirect execution to.