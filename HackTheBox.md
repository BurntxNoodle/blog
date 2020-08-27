---
layout: page
title: HackTheBox
---

![image](https://user-images.githubusercontent.com/41026969/91474771-aaeb1f80-e868-11ea-9bd2-988917019b12.png)

> Hack The Box is an online platform allowing you to test your penetration testing skills and exchange ideas and methodologies with thousands of people in the security field.

## My Writeups

Below is a list of HackTheBox writeups that I have written along with it's general difficulty based on HackTheBox user ratings (noob, easy, medium, hard):

* [Access (easy-medium)](https://securitynoodle.github.io/HackTheBox#HackTheBox-Access)
* [Bashed (noob)]()
* [Beep (easy)]()
* [Blocky (easy)]()
* [Blue (noob)]()
* [Devel (easy)]()
* [Grandpa (easy)]()
* [Granny (easy)]()
* [Jerry (noob-easy)]()
* [Lame (noob)]()
* [Legacy (noob)]()
* [Mirai (easy)]()
* [Netmon (easy)]()
* [Nibbles (easy-medium)]()
* [Obscurity (medium)]()
* [OpenAdmin (easy-medium)]()
* [Optimum (easy)]()
* [Resolute (medium)]()
* [Sense (easy-medium)]()
* [Shocker (easy)]()
* [Tenten (easy)]()

<p align="center">
  <img src="https://user-images.githubusercontent.com/41026969/89838415-2cd50c00-db39-11ea-824b-8ef86b869974.png" />
</p>

### HackTheBox Access

Access is an easy-medium level HackTheBox machine that involves gaining an initial foothold by finding data that's available by a public facing service. Using `mdb-tables`, I was able to export the Microsoft Database entries where afterwards, I did some simple bash scripting to find some clear passwords that allowed me to logon as a low privileged user. To privesc, a `runas /savecred` vulnerability was abused to run an executable as Administrator.   

Read the full writeup [here](https://securitynoodle.github.io/writeups/HTB-Access/).

<p align="center">
  <img src="https://user-images.githubusercontent.com/41026969/89838415-2cd50c00-db39-11ea-824b-8ef86b869974.png" />
</p>

### HackTheBox Bashed

Bashed is an easy-medium level machine from HackTheBox. In this writeup I use dirbuster to look at directories and file names within the website. Knowledge of basic enumeration, and Linux systems should be known; by doing this machine, I learned how to create reverse shells, how to transfer files between an attacker's/victim's machine, and how webshells work.

Read the full writeup [here](https://securitynoodle.github.io/writeups/HTB-Bashed/).

<p align="center">
  <img src="https://user-images.githubusercontent.com/41026969/89838415-2cd50c00-db39-11ea-824b-8ef86b869974.png" />
</p>

### HackTheBox Beep

Beep is a box that requires a good amount of initial enumeration as there are lots of services running on the system. However, knowing how to enumerate each of the servies well and doing some research on the public facing services, one is able to get intial access and later pivot to a root user. This writeup goes over going from no initial access, to getting an initial shell, to getting root. 

Read the full writeup [here](https://securitynoodle.github.io/writeups/HTB-Beep/).

<p align="center">
  <img src="https://user-images.githubusercontent.com/41026969/89838415-2cd50c00-db39-11ea-824b-8ef86b869974.png" />
</p>


### HackTheBox Blocky

Blocky involves learning how using bad password policies and practices can present a vulnerability to a website. This box also simulates a website that is for a Minecraft server, but with that it emulates a young developer's configurations of a web server that's used to escalate privileges to root.

Read the full writeup [here](https://securitynoodle.github.io/writeups/HTB-Blocky/).

<p align="center">
  <img src="https://user-images.githubusercontent.com/41026969/89838415-2cd50c00-db39-11ea-824b-8ef86b869974.png" />
</p>

### HackTheBox Blue 

Blue is a very simplistic machine to exploit which highlights the seriousness of the EternalBlue exploit (thus the name of this challenge). Reading about it [here on the wiki](https://en.wikipedia.org/wiki/EternalBlue) I learned that EternalBlue was used in widespread attacks such as the WannaCry ransomeware attack in 2017. Due to the way SMBv1 in multiple versions of Microsoft Windows handles specifically crafted packets from this attack, it allows remote code execution on the target computer. [CVE link and information here](https://www.cvedetails.com/cve/CVE-2017-0143/)

Read the full writeup [here](https://securitynoodle.github.io/writeups/HTB-Blue/).

<p align="center">
  <img src="https://user-images.githubusercontent.com/41026969/89838415-2cd50c00-db39-11ea-824b-8ef86b869974.png" />
</p>

### HackTheBox Devel

Devel is a HackTheBox machine that involves vulnerability with default configuration of services on a server. By dropping a webshell, I was able to get initial access into the webserver as a low privileged user. By doing more enumeration, certain metasploit modules are found that can be utilized to escalate to System.

Read the full writeup [here](https://securitynoodle.github.io/writeups/HTB-Devel/).

<p align="center">
  <img src="https://user-images.githubusercontent.com/41026969/89838415-2cd50c00-db39-11ea-824b-8ef86b869974.png" />
</p>

### HackTheBox Grandpa

This HackTheBox machine is related to a previous HackTheBox machine: Granny. By achieving root on this box, I learned about exploiting a related CVE that was widely used when released.

Read the full writeup [here](https://securitynoodle.github.io/writeups/HTB-Grandpa/).

<p align="center">
  <img src="https://user-images.githubusercontent.com/41026969/89838415-2cd50c00-db39-11ea-824b-8ef86b869974.png" />
</p>

### HackTheBox Granny

Granny is a HackTheBox challenge that has different methods to achieve root access. The main method of exploiting this system is abusing a commonly seen Webdav upload vulnerability. To complete this box, basic enumeration of services and Windows is all that's needed. Doing this box will teach how to abuse Webdav upload vulnerabilities, and basic Windows privilege escalation techniques. 

Read the full writeup [here](https://securitynoodle.github.io/writeups/HTB-Granny/).

<p align="center">
  <img src="https://user-images.githubusercontent.com/41026969/89838415-2cd50c00-db39-11ea-824b-8ef86b869974.png" />
</p>

### HackTheBox Jerry

Jerry is a challenge involving an apache server running Tomcat (thus the Tom and Jerry reference of the box). Exploiting the box requires basic enumeration scanning, some bruteforcing methods, knowledge of how apache tomcat servers work, and in this particular method that I used to solve: msfvenom and metasploit to create a metrepeter sessions. All of which is documented below.

Read the full writeup [here](https://securitynoodle.github.io/writeups/HTB-Jerry/).

<p align="center">
  <img src="https://user-images.githubusercontent.com/41026969/89838415-2cd50c00-db39-11ea-824b-8ef86b869974.png" />
</p>

### HackTheBox Lame

HTB Lame is a starter level machine that can be easily exploited using metasploit and some googling. Samba SMB (server message block) is a vulnerable process that in this challenge, was used to get root access into the machine using a metasploit module. To exploit this box, you just need to know basic enumeration, linux services, and some googling.

Read the full writeup [here](https://securitynoodle.github.io/writeups/HTB-Lame/). 

<p align="center">
  <img src="https://user-images.githubusercontent.com/41026969/89838415-2cd50c00-db39-11ea-824b-8ef86b869974.png" />
</p>

### HackTheBox Legacy

Legacy is a challenge that is fairly simplistic, uncomplicated challenge. Doing this challenge highlights the security vulnerabilities that are associated with SMB on Windows. To do this challenge, the skills required isn't too high, one just needs to know how to enumerate the box/network and basic windows knowledge. Doing this challenge will develop skills such as identifying services that are vulnerable to attacks, how to carry out said attacks, and any tools used.

Read the full writeup [here](https://securitynoodle.github.io/writeups/HTB-Legacy/).

<p align="center">
  <img src="https://user-images.githubusercontent.com/41026969/89838415-2cd50c00-db39-11ea-824b-8ef86b869974.png" />
</p>

### HackTheBox Mirai

Mirai is a different type of HackTheBox machine as it's in my opinion, more CTF like than pentesting becaise it also includes some forensics/reverse engineering work. The origins of this box is interesting as the name referes to a botnet named Mirai that was able to spread onto other systems and infect them to grow the botnet.

Read the full writeup [here](https://securitynoodle.github.io/writeups/HTB-Mirai/).

<p align="center">
  <img src="https://user-images.githubusercontent.com/41026969/89838415-2cd50c00-db39-11ea-824b-8ef86b869974.png" />
</p>

### HackTheBox Netmon

Netmon teaches about vulnerabiities associated with the PRTG Network Monitor. Exploiting this box requires some enumeration knowledge of windows, FTP, some knowledge of PRTG network monitor (lots of googling was done during this challenge), and a little bit of knowledge of powershell. In the method that I used to exploit netmon (that will be shown on this writeup), I also used (and learned how to use by doing this challenge) SimpleHTTPServer on kali linux.

Read the full writeup [here](https://securitynoodle.github.io/writeups/HTB-Netmon/).

<p align="center">
  <img src="https://user-images.githubusercontent.com/41026969/89838415-2cd50c00-db39-11ea-824b-8ef86b869974.png" />
</p>

### HackTheBox Nibbles

Nibbles is a box that is running a `nibbleblog` site which has a known exploit with an associated metasploit module. Though for the exploit module to work, credentials are needed. By enumerating the web directories on the system, a `users.xml` file was found that showed in plaintext an administrator account. From there, I was able to guess a relatively easy password to gain initial access onto the system. PrivEsc involved finding binaries that the user we logged onto as can run without a password.

Read the full writeup [here](https://securitynoodle.github.io/writeups/HTB-Nibbles/).

<p align="center">
  <img src="https://user-images.githubusercontent.com/41026969/89838415-2cd50c00-db39-11ea-824b-8ef86b869974.png" />
</p>

### HackTheBox Obscurity

Obscurity is a medium level HackTheBox machine that features a customized web server, and a couple of other python files that we have to do some reverse engineering to understand it, to later develop solutions to circumvent security holes.

Here are some key points of the box that leads to
getting root access:
* Finding the SuperSecretServer python file.
* Finding the security hole in the python file.
* Crafting the payload to get initial shell access.
* Finding out how the SuperSecretCrypt python file works.
* Calculating/Writing a program to do the math and find the key.
* Understanding that the BetterSSH file writes a temporary file and quickly deletes it.
* Finding a way to retrieve the file before it gets deleted.
* Knowing how to decrypt the hashed root password.

Read the full writeup [here](https://securitynoodle.github.io/writeups/HTB-Obscurity.pdf).

<p align="center">
  <img src="https://user-images.githubusercontent.com/41026969/89838415-2cd50c00-db39-11ea-824b-8ef86b869974.png" />
</p>

### HackTheBox OpenAdmin

OpenAdmin is a box with an easy to medium level process of obtaining the user credentials, into which gaining the root shell is a matter of research and simple enumeration. The box features two users: Jimmy and Joanna that we had to get access into to eventually elevate to root. The main components of gaining unauthorized root access is as follows:

* Public facing of an OpenNetAdmin interface to the public. This also includes showing version number and having an outdated version being used.
* Plaintext credentials found in `/opt/ona/www/local/config/database_settings.inc.php`.
* Password reuse of user Jimmy.
* Plaintext index file showing functionality, a clear hash with encoding algorithm is also shown.
* Private RSA key is shared between users.
* User Joanna can do a sudo command without supplying a password.


Read the full writeup [here](https://securitynoodle.github.io/writeups/HTB-OpenAdmin.pdf).

<p align="center">
  <img src="https://user-images.githubusercontent.com/41026969/89838415-2cd50c00-db39-11ea-824b-8ef86b869974.png" />
</p>

### HackTheBox Optimum

This HackTheBox machine involves exploiting a known vulnerability in HFS - HTTP File Server that is public facing. In addition, to escalate privileges, I used other tools such as `windows-exploit-suggester.py` to do further enumeration.

Read the full writeup [here](https://securitynoodle.github.io/writeups/HTB-Optimum/).

<p align="center">
  <img src="https://user-images.githubusercontent.com/41026969/89838415-2cd50c00-db39-11ea-824b-8ef86b869974.png" />
</p>

### HackTheBox Resolute

Resolute is a Windows box that features two users where the first user is used to get initial access (Melanie) and while as Melanie, one is supposed to find plaintext credentials into the second user’s account: Ryan. It is discovered that Ryan is under the Contractors group which is under the DnsAdmins group. This information is used to escalate into Administrator access via dll injection. The key components of gaining unauthorized administrator access:

* Default/Reuse of new accounts that are created
* Plaintext credentials of other users are found under the “PSTranscripts” directory
* Incorrect group settings/policies

Read the full writeup [here](https://securitynoodle.github.io/writeups/HTB-Resolute.pdf).

<p align="center">
  <img src="https://user-images.githubusercontent.com/41026969/89838415-2cd50c00-db39-11ea-824b-8ef86b869974.png" />
</p>

### HackTheBox Sense

Sense is a medium level box that requires some basic knowledge of PHP and enumeration. By achieving root access on this box, I learned how to exploit PFsense by bypassing some filtering, and using a pubic exploit proof of concept.

Read the full writeup [here](https://securitynoodle.github.io/writeups/HTB-Sense/).

<p align="center">
  <img src="https://user-images.githubusercontent.com/41026969/89838415-2cd50c00-db39-11ea-824b-8ef86b869974.png" />
</p>

### HackTheBox Shocker

By abusing a known exploit named [Shellshock](https://en.wikipedia.org/wiki/Shellshock_(software_bug)), I was able to get initial access onto HackTheBox's Shocker machine. For this, I used the metasploit exploit module `apache_mod_cgi_bash_env_exec`. Privilege escalation was pretty simple, did more enumeration to find suggested metasploit modules which one ended up privilege escalating my permissions to root.

Read the full writeup [here](https://securitynoodle.github.io/writeups/HTB-Shocker/).

<p align="center">
  <img src="https://user-images.githubusercontent.com/41026969/89838415-2cd50c00-db39-11ea-824b-8ef86b869974.png" />
</p>

### HackTheBox Tenten

Tenten is a box that is running a web server that has wordpress. By doing this box, I learned how to enumerate wordpress and how to exploit outdated plugins. From here, this box is more CTF-like as there's an image with an id-rsa key hidden in it. For this part I used steghide and simply cracked it to find the password.

Read the full writeup [here](https://securitynoodle.github.io/writeups/HTB-Tenten/).

<p align="center">
  <img src="https://user-images.githubusercontent.com/41026969/89838415-2cd50c00-db39-11ea-824b-8ef86b869974.png" />
</p>

