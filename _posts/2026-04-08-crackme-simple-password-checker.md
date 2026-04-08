---
layout: post
title: "Crackme Writeup: Simple Password Checker"
date: 2026-04-08
---
## Challenge

A simple password checker that checks against a hardcoded string.

## Approach

First I made the binary executable:

```bash
chmod +x simp-password
```

Then tried inputting a few characters to see what response I'd get.

![wrong password](/assets/images/wrong-password.png)

As you can see, inputting the wrong password gives "Wrong try again".

Next I opened Ghidra and looked at the functions list.

![functions](/assets/images/functions.png)

Most functions aren't interesting — main is the one to look at.

![main decompiled](/assets/images/main.png)

The binary asks for a password, stores your input in a variable, 
then compares it to the hardcoded string `"iloveicecream"` using 
strcmp. If strcmp returns 0 (match), it prints "i love ice cream too", 
otherwise "Wrong try again".

## Solution

The hardcoded password was `iloveicecream` found directly in the 
strcmp call in main.

![success](/assets/images/success.png)

