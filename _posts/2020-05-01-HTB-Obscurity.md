---
layout: post
title: HackTheBox Obscurity Writeup
---

Obscurity is a medium level HackTheBox machine that features a customized web server, and a couple of other python files that we have to do some reverse engineering to understand it, to later develop solutions to circumvent security holes.

Here are some key points of the box that leads to
getting root access:
* Finding the SuperSecretServer python file.
* Finding the security hole in the python file.
* Crafting the payload to get initial shell access.
* Finding out how the SuperSecretCrypt python file works.
* Calculating/Writing a program to do the math and find the key.
* Understanding that the BetterSSH file writes a temporary file and quickly deletes it.
* Finding a way to retrieve the file before it gets deleted.
* Knowing how to decrypt the hashed root password.

Read the full writeup [here](https://burntxnoodle.github.io/writeups/HTB-Obscurity/).
