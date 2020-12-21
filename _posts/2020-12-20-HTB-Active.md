---
layout: post
title: New HackTheBox Active Writeup
---

HackTheBox Active is a fairly easy box but still goes over a topic that can be complex which is Windows Active Directory. By doing this box, I was first able to get more comfortable doing recon on Windows environments (more specifically enumerating SMB). During the privilege escalation part, I learned the basics of how Active Directory works, along with one way of exploiting its functionality. Doing so, I got experience with [Impacket](https://github.com/SecureAuthCorp/impacket) and a couple modules that are included within it.

The key components to exploiting this box are as follows:

- First realizing that SMB is open on ports 139/445
- Finding out the Replication share can be accessed anonymously, allowing to view the contents without any credentials
- Obtaining the ```Groups.xml``` file which contains an AES encrypted password
- Decrypting the password allows to login with a valid set of credentials
- Finding out that a service ticket can be obtained due to having a valid set of credentials (Kerberoast)
- Obtaining service ticket gets a hash that can be cracked offline using John the Ripper
- Once the hash is cracked, using [Impacket’s](https://github.com/SecureAuthCorp/impacket) ```wmiexec.py``` or ```psexec.py``` can be used to access the system as Administrator

Read the full writeup [here on my medium blog](https://securitynoodle.medium.com/hackthebox-active-writeup-57f294424435)!