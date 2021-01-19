---
layout: post
title: Using Volatility within SANS SIFT workstation to do memory forensics - HackTheBox Export (Forensics Challenge) Writeup
---

Export is a HackTheBox challenge that is under their forensics list. For this challenge, I was given a .raw file which is a memory dump of a system in which memory forensics was done to figure out what is going on during the time the dump was created. By utilizing the memory forensics tool Volatility, I was able to get information about the processes running on the system where I saw cmd.exe being used. Doing further analysis around what was done using the command line, the flag was found.

Read my full writeup [here](https://securitynoodle.medium.com/hackthebox-export-forensics-challenge-writeup-1cbb6c0be4fd).