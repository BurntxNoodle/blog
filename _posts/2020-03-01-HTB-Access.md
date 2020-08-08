---
layout: post
title: HackTheBox Access Writeup
---

Access is an easy-medium level HackTheBox machine that involves gaining an initial foothold by finding data that's available by a public facing service. Using `mdb-tables`, I was able to export the Microsoft Database entries where afterwards, I did some simple bash scripting to find some clear passwords that allowed me to logon as a low privileged user. To privesc, a `runas /savecred` vulnerability was abused to run an executable as Administrator.   

Read the full writeup [here](https://burntxnoodle.github.io/writeups/HTB-Access/).
