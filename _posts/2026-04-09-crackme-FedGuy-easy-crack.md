---
layout: post
title: "Crackme Writeup: FedGuy easy crack"
date: 2026-04-09
link: "https://crackmes.one/crackme/691abffc2d267f28f69b7e8a"
---



## Challenge

A password checker binary. The goal is to find the correct password.

## Approach

**Step 1 — Initial recon**

First, make the binary executable and run it to see what we're dealing with. Inputting a few random characters gives us `"You lose :("` 

![fail](/assets/images/2026-04-09/fail.png)


**Step 2 — Opening Ghidra**

Loading the binary into Ghidra and checking the functions list, two stand out: `main` and `bytes_to_hex`.

![functions](/assets/images/2026-04-09/function-tree.png)

**Step 3 — Reversing main**

Decompiling `main` reveals the following logic:

1. Prompts for a password through `scanf`
2. Calculates the length of the input with `strlen`
3. Passes the input to `SHA256()`, storing the raw hash in `local_78`
4. Passes `local_78` to `bytes_to_hex()`, which writes a string into `local_58`
5. Compares `local_58` against the hardcoded string:

`811eb81b9d11d65a36c53c3ebdb738ee303403cb79d781ccf4b40764e0a9d12a`

![main](/assets/images/2026-04-09/main-func-decomp.png)

**Step 4 — Investigating SHA256**

Double clicking `SHA256` in Ghidra shows no real code — just a warning about bad instruction data and the comment `/* SHA256@OPENSSL_3.0.0 */`. My guess is that its using the systems OpenSSL library It's standard SHA256  but Ghidra just can't decompile an external library call.
   
![sha2556](/assets/images/2026-04-09/sha-256.png)


**Step 5 — Investigating bytes_to_hex**

This function takes the raw 32-byte SHA256 output and converts it to a 64-character  hex string using `sprintf` with `"%02x"` per each byte. 

![bytes_to_hex](/assets/images/2026-04-09/bytes-to-hex.png)



**Step 6 — Cracking the hash**

Since SHA256 can't be reversed, we use hashcat against a common wordlist:

```bash
hashcat -m 1400 hash.txt 1000000-password-seclists.txt
```

- `-m 1400` — SHA-256 mode
- `hash.txt` — file containing the extracted hash
- `1000000-password-seclists.txt` — common passwords wordlis

![hashcat](/assets/images/2026-04-09/cracked-hash.png)

## Solution

The binary hashes user input with SHA256 and compares it against a hardcoded value. Cracking the hash with hashcat reveals the password is:

```
chicken
```

![solved](/assets/images/2026-04-09/solved.png)


