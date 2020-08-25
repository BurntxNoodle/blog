---
layout: post
title: HackTheBox Shocker Writeup
---

By abusing a known exploit named [Shellshock](https://en.wikipedia.org/wiki/Shellshock_(software_bug)), I was able to get initial access onto HackTheBox's Shocker machine. For this, I used the metasploit exploit module `apache_mod_cgi_bash_env_exec`. Privilege escalation was pretty simple, did more enumeration to find suggested metasploit modules which one ended up privilege escalating my permissions to root.

Read the full writeup [here](https://securitynoodle.github.io/writeups/HTB-Shocker/).
