---
layout: post
title: "Crackme Writeup: Izijerrys Int Overflow"
date: 2026-04-12
---



## Challenge

A simple password checker binary. The goal is to find the correct password.

## Approach

**Step 1 — Initial recon**

First, make the binary executable and run it to see what we're dealing with. Inputting a few random characters gives us `"Incorrect Password!!!"`

![fail](/assets/images/2026-04-12/fail.png)

**Step 2 — Opening Ghidra**

Loading the binary into Ghidra and checking the functions list, two stand out: `main` and `GetPass`.

![function tree](/assets/images/2026-04-12/function-tree.png)

**Step 3 — Reversing main**

Decompiling `main` reveals the following logic:

1. Call `GetPass()` and store the return value in `__nptr`
2. Use `atoi()` to convert the string to an integer
3. If the value equals `-0x2023e3c2`, print `"Password Correct!!!"`

![main decompiled](/assets/images/2026-04-12/main.png)

**Step 4 — Investigating GetPass**

`GetPass` doesn't do anything other than ask for input via `scanf` and return it.

![getpass](/assets/images/2026-04-12/getpass.png)

## Solution

We need `atoi()` to return `-0x2023e3c2`. Converting that to decimal:

```python
>>> -0x2023e3c2
-539222978
```

Entering `-539222978` as the password returns `"Password Correct!!!"`.

![solved](/assets/images/2026-04-12/success.png)

