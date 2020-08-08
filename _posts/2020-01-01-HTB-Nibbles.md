---
layout: post
title: HackTheBox Nibbles Writeup
---

Nibble is a box that is running a `nibbleblog` site which has a known exploit with an associated metasploit module. Though for the exploit module to work, credentials are needed. By enumerating the web directories on the system, a `users.xml` file was found that showed in plaintext an administrator account. From there, I was able to guess a relatively easy password to gain initial access onto the system. PrivEsc involved finding binaries that the user we logged onto as can run without a password.

Read the full writeup [here](https://burntxnoodle.github.io/writeups/HTB-Nibbles/).