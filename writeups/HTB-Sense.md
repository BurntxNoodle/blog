---
layout: page
title: HackTheBox Sense Writeup
---

Sense is a medium level box that requires some basic knowledge of PHP and enumeration. By achieving root access on this box, I learned how to exploit PFsense by bypassing some filtering, and using a pubic exploit proof of concept.

### Enumeration 

First I did a nmap scan: ```nmap -sC -sV -oA scan 10.10.10.60```. This will scan for versioning and open ports and will output the results to ```scan.nmap```. Here's what the results produced.

![image](https://user-images.githubusercontent.com/41026969/66668918-5aed7500-ec24-11e9-9d96-fbc62d353f6a.png)

Since we see that ports ```80``` and ```443``` are open, we know the server is running an HTTP/HTTPS server. Let's check the website:

![image](https://user-images.githubusercontent.com/41026969/66669059-ae5fc300-ec24-11e9-90a0-330dde735879.png)

We see a pfsense login page (which is why this box is named Sense). I tried some the normal default credits of ```admin```:```pfsense``` and some common user/password combinations and got no luck.

Next, I use dirbuster to potentially find any directories and files within the webserver. I start off with the ```common.txt``` wordlist. And here's what it outputted

![image](https://user-images.githubusercontent.com/41026969/66670083-e7009c00-ec26-11e9-9aa4-549eefb1bbb4.png)

After checking through each of the files for awhile, I couldn't find any useful information. I decide to use another wordlist with additional extensions: ```txt, html, jpeg``` (on dirbuster near the lower right there's a ```File extension``` box, you can add extensions by separating them with a comma). 

Here are the results:

![image](https://user-images.githubusercontent.com/41026969/66670581-2085d700-ec28-11e9-9024-e3bbdfddb7a5.png)

There are couple interesting files: ```changelog.txt``` and ```system-users.txt```. 

Here's what the ```changelog.txt``` file says:

![image](https://user-images.githubusercontent.com/41026969/66674065-08b25100-ec30-11e9-8c89-c61c4f893cab.png)

So in this txt file we see only 2 out of 3 vulnerabilities were patched... Interesting

Here's what the ```system-users.txt``` file says:

![image](https://user-images.githubusercontent.com/41026969/66685978-f47c4d00-ec4b-11e9-9d2d-0cbff3c20860.png)

### Exploitation 

So we have some information about an unpatched vulnerability within pfsense that the firewall failed to update. 

Doing some research and googling ```pfsense vulnerabilities```, I come across [this page that lists published vulnerabilities within pfsense](https://www.cvedetails.com/vulnerability-list/vendor_id-11749/product_id-21763/Pfsense-Pfsense.html)

So with the hint we were given about 2 out of 3 vulnerabilities, I assume the most recent one from 2019 wasn't accounted for as this could be an older box or version.

Reading through each of the vulnerabilities, [CVE-2016-10709](https://www.cvedetails.com/cve/CVE-2016-10709/) seemed the most likely to work. Reading through the [exploit-db](https://www.exploit-db.com/exploits/39709), this is how they described the vulnerability

```
The status_rrd_graph_img.php page is vulnerable to command injection via
the graph GET parameter. A non-administrative authenticated attacker
having access privileges to the graph status functionality can inject
arbitrary operating system commands and execute them in the context of
the root user. Although input validation is performed on the graph
parameter through a regular expression filter, the pipe character is not
removed. Octal characters sequences can be used to encode a payload,
bypass the filter for illegal characters, and create a PHP file to
download and execute a malicious file (i.e. reverse shell) from a remote
attacker controlled host.
```

So we see that as an non admin user, we can remotely execute code as root. 

From the hint earlier, we know there was a ticket for an account ```Rohit``` to be made with default passwords. Testing on the main webpage, I figured out the ```user:pass``` is ```rohit:pfsense```. (note: it has to be lower case 'r' for whatever reason and I was stuck on this for longer that I should have been).

Reading back on the cvedetails page linked above, I read there's a [rapid7 link](https://www.rapid7.com/db/modules/exploit/unix/http/pfsense_graph_injection_exec) along with information about a metasploit module we can use.

The metasploit module: ```exploit/unix/http/pfsense_graph_injection_exec```.

With the information we have gathered, we can run this metasploit module. Here's a picture of me running the metasploit module and configuring the settings:

![image](https://user-images.githubusercontent.com/41026969/66685690-3789f080-ec4b-11e9-8490-f7a69368a6e5.png)

Running the exploit, we get a meterpreter session! Doing a ```whoami``` check reveals we are ```root```! 

![image](https://user-images.githubusercontent.com/41026969/66685827-86d02100-ec4b-11e9-8f5a-8e041bc35777.png)

### Flags:

User: ```/home/rohit/user.txt```

Root: ```/root/root.txt```