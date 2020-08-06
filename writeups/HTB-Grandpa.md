---
layout: page
title: HackTheBox Grandpa Writeup
---

This HackTheBox machine is related to a previous HackTheBox machine: Granny. By achieving root on this box, I learned about exploiting a related CVE that was widely used when released.

### Enumeration 

Starting off with a basic enumeration scan: ```nmap -sC -sV -oA granny 10.10.10.14```. Here's the result:

![image](https://user-images.githubusercontent.com/41026969/71902626-76d3ba80-3130-11ea-853b-801a67433d8a.png)

We see that port ```80``` is open that's hosting an IIS webserver. Doing a previous challenge [Granny](https://github.com/BurntxNoodle/PenetrationTesting/tree/master/HTB%20-%20Granny) I learned about WebDAV and this box also seems to have some WebDAV funcitonality. I do a similar ```davtest``` scan. Here's the result:

![image](https://user-images.githubusercontent.com/41026969/71902458-1fcde580-3130-11ea-866f-baef0c1043b2.png)

Meanwhile, in the background, I run a ```gobuster``` script to see if there's any additional directories. The command I did was ```gobuster dir -u http://10.10.10.14 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt```. And this is what it found:

```
/images (Status: 301)
/Images (Status: 301)
/IMAGES (Status: 301)
/_private (Status: 403)
```

```Status: 403``` means we don't have access to that directory, ```Status: 301``` is a URL redirection, but we can access it.

So basically from our findings, we see the only open port is ```80``` with WebDAV. But with this WebDAV (unlike HackTheBox grandma), we cannot upload files so we can't upload an ```.asp``` file to execute and get a reverse shell. 

I decided to do some googling:

![image](https://user-images.githubusercontent.com/41026969/71909350-dd130a00-313d-11ea-9dde-2abd2d26ab4f.png)

[The first link](https://www.rapid7.com/db/modules/exploit/windows/iis/iis_webdav_scstoragepathfromurl) by Rapid7 shows the module ```exploit/windows/iis/iis_webdav_scstoragepathfromurl```. 

### Exploitation

Using the module, configuring the settings, then later executing the exploit, we get a meterpreter shell.

![image](https://user-images.githubusercontent.com/41026969/71910480-fc129b80-313f-11ea-879e-02efc25368e2.png)

In meterpreter, I do ```shell``` to get a native windows shell. Doing a ```whoami``` check outputs ```nt authority\network service```. This isn't the administrator account.

Next, something I learned by doing this HackTheBox challenge is to migrate processes to a different process that you own or a process that's the same architecture as your exploit/payload. So for this challenge, I migrate to a process that I own (as ```nt authority\network service```. 

*NOTE: When doing this challenge, the meterpreter session kept closing randomly. I think it's due to the process that I migrated to so I suggest trying a different process if possible.

![image](https://user-images.githubusercontent.com/41026969/71935472-da7fd700-3174-11ea-9c30-ffc5595828b3.png)

### Privesc

First step I do is to use a metasploit module that will collect a list of known exploits based on the system information you have on the machine and then suggests you a list of exploits that may work. 

```
msf5 exploit(windows/iis/iis_webdav_scstoragepathfromurl) > search suggester

Matching Modules
================

   #  Name                                      Disclosure Date  Rank    Check  Description
   -  ----                                      ---------------  ----    -----  -----------
   0  post/multi/recon/local_exploit_suggester                   normal  No     Multi Recon Local Exploit Suggester


msf5 exploit(windows/iis/iis_webdav_scstoragepathfromurl) > use post/multi/recon/local_exploit_suggester 
msf5 post(multi/recon/local_exploit_suggester) > show options

Module options (post/multi/recon/local_exploit_suggester):

   Name             Current Setting  Required  Description
   ----             ---------------  --------  -----------
   SESSION                           yes       The session to run this module on
   SHOWDESCRIPTION  false            yes       Displays a detailed description for the available exploits

msf5 post(multi/recon/local_exploit_suggester) > set SESSION 1
SESSION => 1
msf5 post(multi/recon/local_exploit_suggester) > run

[*] 10.10.10.14 - Collecting local exploits for x86/windows...
[*] 10.10.10.14 - 29 exploit checks are being tried...
[+] 10.10.10.14 - exploit/windows/local/ms10_015_kitrap0d: The service is running, but could not be validated.
[+] 10.10.10.14 - exploit/windows/local/ms14_058_track_popup_menu: The target appears to be vulnerable.
[+] 10.10.10.14 - exploit/windows/local/ms14_070_tcpip_ioctl: The target appears to be vulnerable.
[+] 10.10.10.14 - exploit/windows/local/ms15_051_client_copy_image: The target appears to be vulnerable.
[+] 10.10.10.14 - exploit/windows/local/ms16_016_webdav: The service is running, but could not be validated.
[+] 10.10.10.14 - exploit/windows/local/ms16_032_secondary_logon_handle_privesc: The service is running, but could not be validated.
[+] 10.10.10.14 - exploit/windows/local/ms16_075_reflection: The target appears to be vulnerable.
[+] 10.10.10.14 - exploit/windows/local/ms16_075_reflection_juicy: The target appears to be vulnerable.
[+] 10.10.10.14 - exploit/windows/local/ppr_flatten_rec: The target appears to be vulnerable.
[*] Post module execution completed
msf5 post(multi/recon/local_exploit_suggester) > sessions
```

Trying the first exploit listed (```exploit/windows/local/ms10_015_kitrap0d```) that "appears to be vulnerable" by ended up crashing the box so that didn't work.

Trying the second exploit (```exploit/windows/local/ms14_070_tcpip_ioctl```) that "appears to be vulnerable" by ended up getting me a shell:

![image](https://user-images.githubusercontent.com/41026969/71937572-c4751500-317a-11ea-8da6-ff37d0e6ae50.png)

### Getting Root/User flags

Eventually I wander into this directory that contains an ```Administrator``` directory and a ```Harry``` directory that looks to be a user on the box.

![image](https://user-images.githubusercontent.com/41026969/71938216-b7f1bc00-317c-11ea-9642-ff898efbab9f.png)

Checking the ```Desktop``` inside each of those reveals the Root flag and the User flag

### Flags

Root: ```C:\Documents and Settings\Administrator\Desktop\root.txt```

User: ```C:\Documents and Settings\Harry\Desktop\user.txt```

### Sources
- [Rapid7 module for initial shell](https://www.rapid7.com/db/modules/exploit/windows/iis/iis_webdav_scstoragepathfromurl)
- [HackTheBox Granny Writeup (also deals with IIS/WebDAV](https://github.com/BurntxNoodle/PenetrationTesting/tree/master/HTB%20-%20Granny)
