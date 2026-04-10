---
layout: post
title: "Crackme Writeup: YandereMia's Introduction to RE"
date: 2026-04-10
---


## Challenge

A simple password checker binary. The goal is to find the correct password.

## Approach

**Step 1 — Initial recon**

First, make the binary executable and run it to see what we're dealing with. Inputting a few random characters gives us `"[ERROR] Password is incorrect"`

![fail](/assets/images/2026-04-10/fail.png)

**Step 2 — Opening Ghidra**

Loading the binary into Ghidra and checking the functions list, only one stands out: `main` 

![functions](/assets/images/2026-04-10/func-tree.png)

**Step 3 — Reversing main**

Decompiling `main` reveals the following logic:

1. Prompts for a password through `scanf`
2. Compares input using `strcmp` to this string:
	`7Wtyr`
3. If both are equal print  `"[OK] Passowrd Found"`

![main](/assets/images/2026-04-10/main.png)

## Solution

The binary takes user input and compares it against the hard coded value: `7Wtyr` when correct you get the success message: `"[OK] Passowrd Found"`

![success](/assets/images/2026-04-10/success.png)
