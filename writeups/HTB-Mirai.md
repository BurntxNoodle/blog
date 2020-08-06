---
layout: page
title: HackTheBox Mirai Writeup
---

Mirai is a different, but interesting type of HackTheBox machine as it's in my opinion, more CTF like than pentesting becaise it also includes some forensics/reverse engineering work. The origins of this box is interesting as the name referes to a botnet named Mirai that was able to spread onto other systems and infect them to grow the botnet.

### Enumeration 

To start off, I do: ```nmap -sC -sV -oA mirai 10.10.10.48```. Here's what was outputted:

![image](https://user-images.githubusercontent.com/41026969/72224053-fa801380-3543-11ea-9c41-f60a7712eb4a.png) 

There are a few ports open:
- Port 22 which is an SSH server. Currently we have no creds we can try.
- Port 53 which is a DNS server. Doing some research [about DNS](https://www.grc.com/port_53.htm): ```Port 53 is used by the Domain Name System (DNS), a service that turns human readable names into IP addresses that the computer understands.``` More specifically about [dnsmasq](https://en.wikipedia.org/wiki/Dnsmasq): ```Dnsmasq provides Domain Name System (DNS) forwarder, Dynamic Host Configuration Protocol (DHCP) server, router advertisement and network boot features for small computer networks, created as free software.```
- Port 80 which is a webserver.

Checking out the main page of the website: 

![image](https://user-images.githubusercontent.com/41026969/72390779-8f783d80-36f9-11ea-9893-afa87bcda2bc.png)

We can verify that the ```nmap``` scan is correct the main homepage is a blank HTML page.

Next step is to run a ```gobuster``` scan to find any potential directories: ```gobuster dir -u http://10.10.10.48 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt```. 

(Explanation of the code above): ```gobuster``` will run the application. ```dir``` is the mode that we'll be using gobuster for (it will brute force directories). ```-u``` is the flag that the next string is the URL. ```-w``` is the flag that the next string is the wordlist that we'll be using.

Here is the output of running that gobuster scan:

```
===============================================================
Gobuster v3.0.1
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@_FireFart_)
===============================================================
[+] Url:            http://10.10.10.48
[+] Threads:        10
[+] Wordlist:       /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Status codes:   200,204,301,302,307,401,403
[+] User Agent:     gobuster/3.0.1
[+] Timeout:        10s
===============================================================
2020/01/08 18:25:05 Starting gobuster
===============================================================
/admin (Status: 301)
/versions (Status: 200)
===============================================================
2020/01/08 18:39:02 Finished
===============================================================
```

```gobuster``` discovered two URLs:

- ```http://10.10.10.48/admin```
- ```http://10.10.10.48/versions```

Visiting the ```/admin``` directory:

![image](https://user-images.githubusercontent.com/41026969/72391426-11b53180-36fb-11ea-987e-359606d26de5.png)

We see there's a webpage that has something called ```Pi-hole```. 

From the [Pi-hole wiki page](https://en.wikipedia.org/wiki/Pi-hole): ```Pi-hole is a Linux network-level advertisement and Internet tracker blocking application which acts as a DNS sinkhole (and optionally a DHCP server), intended for use on a private network. It is designed for use on embedded devices with network capability, such as the Raspberry Pi, but it can be used on other machines running Linux and cloud implementations.```

So from this page, it is learned that Pi-hole does advertisement and internet tracker blocking on a private network. It is also learned that it's designed for devices like a Raspberry Pi. 

### Getting a shell

Doing some research I come across a [an article](https://itsfoss.com/ssh-into-raspberry/) with SSH info into a Raspberry Pi. It says the default credentials are 

```
user: pi
password: raspberry
```

Testing out those credentials:

![image](https://user-images.githubusercontent.com/41026969/72392352-d5370500-36fd-11ea-93b8-75e12eec85a4.png)

We get a shell.

Doing an ```id``` check reveals we aren't a root user. 

Wandering out and eventually checking out the ```Desktop``` directory, we get the user flag:

![image](https://user-images.githubusercontent.com/41026969/72393136-03b5df80-3700-11ea-9001-088f04fbb5d1.png)

Though doing a ```sudo -l``` command to list commands that are allowed on our current user. Doing so results in this:

```
pi@raspberrypi:~ $ sudo -l
Matching Defaults entries for pi on localhost:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin

User pi may run the following commands on localhost:
    (ALL : ALL) ALL
    (ALL) NOPASSWD: ALL
```

This means we can run ```sudo su - ``` to switch to the ```root``` user without a password.

![image](https://user-images.githubusercontent.com/41026969/72392860-1a0f6b80-36ff-11ea-9a16-9781f0de4d5e.png)

Doing a ```ls -la``` check, there's a ```root.txt``` flag... But ```cat root.txt``` outputs the following message:

```
root@raspberrypi:~# ls
root.txt
root@raspberrypi:~# cat root.txt 
I lost my original root.txt! I think I may have a backup on my USB stick...  
```

To check the devices (things that can read/write files like a hard drive or USB stick) we can do ```lsblk``` which stands for list blocks. Here's what it shows:

```
root@raspberrypi:~# lsblk
NAME   MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sda      8:0    0   10G  0 disk 
├─sda1   8:1    0  1.3G  0 part /lib/live/mount/persistence/sda1
└─sda2   8:2    0  8.7G  0 part /lib/live/mount/persistence/sda2
sdb      8:16   0   10M  0 disk /media/usbstick
sr0     11:0    1 1024M  0 rom  
loop0    7:0    0  1.2G  1 loop /lib/live/mount/rootfs/filesystem.squashfs
```

We can also do ```df``` to see more information about the disks. [Man page here](https://linux.die.net/man/1/df)

```
root@raspberrypi:~# df
Filesystem     1K-blocks    Used Available Use% Mounted on
aufs             8856504 2832120   5551452  34% /
tmpfs             102396    4868     97528   5% /run
/dev/sda1        1354528 1354528         0 100% /lib/live/mount/persistence/sda1
/dev/loop0       1267456 1267456         0 100% /lib/live/mount/rootfs/filesystem.squashfs
tmpfs             255988       0    255988   0% /lib/live/mount/overlay
/dev/sda2        8856504 2832120   5551452  34% /lib/live/mount/persistence/sda2
devtmpfs           10240       0     10240   0% /dev
tmpfs             255988       8    255980   1% /dev/shm
tmpfs               5120       4      5116   1% /run/lock
tmpfs             255988       0    255988   0% /sys/fs/cgroup
tmpfs             255988       8    255980   1% /tmp
/dev/sdb            8887      93      8078   2% /media/usbstick
tmpfs              51200       0     51200   0% /run/user/999
tmpfs              51200       0     51200   0% /run/user/1000
```

We see there's a 10MB disk located at ```/media/usbstick```.

Checking it out, there's another message in ```damnit.txt```:

```
root@raspberrypi:~# cd /media/usbstick/
root@raspberrypi:/media/usbstick# ls -la
total 18
drwxr-xr-x 3 root root  1024 Aug 14  2017 .
drwxr-xr-x 3 root root  4096 Aug 14  2017 ..
-rw-r--r-- 1 root root   129 Aug 14  2017 damnit.txt
drwx------ 2 root root 12288 Aug 14  2017 lost+found
root@raspberrypi:/media/usbstick# cat damnit.txt 
Damnit! Sorry man I accidentally deleted your files off the USB stick.
Do you know if there is any way to get them back?

-James
```

### Recovering the root flag

Doing some research, I found out [on this stack exchange question](https://unix.stackexchange.com/questions/2677/recovering-accidentally-deleted-files) it is found out that you can recover some deleted text from a device. Basically we can read the filesystem to see if there's anything that was overwritten/deleted.

From the ```df``` and the ```lsblk``` commands, we see that the usb filesystem is on ```/dev/sdb```

Doing a similar approach to the stack exchange question, I did ```strings /dev/sdb``` to view the printable characters in the file. Doing so outputs something that resembles a flag:

![image](https://user-images.githubusercontent.com/41026969/72542386-104a4d00-3852-11ea-946e-046c5baa37b3.png)

Inputting that hash reveals that it is indeed the root flag!

Another alternate way to recover the file that I learned from [IppSec's video writeup](https://www.youtube.com/watch?v=SRmvRGUuuno) is that we can utilize ```xxd``` to view the hex dump of the device.

Though since it's deleted and mostly overwritten by a bunch of 0's such as:

```
0069040: 0000 0000 0000 0000 0000 0000 0000 0000  ................
```

It will output a bunch of similar lines. Thus we can pipe (```|```) the output against ```grep``` to remove those lines that were overwritten with 0's.

Here's the command I did: ```root@raspberrypi:/media/usbstick# xxd /dev/sdb | grep -v "0000 0000 0000 0000 0000 0000 0000 0000"```

(Explanation of the command above): ```xxd /dev/sdb``` to view the hex dump. Throw the output to grep. From the [grep man page](https://linux.die.net/man/1/grep) we see we can use ```-v``` to invert match: ```Invert the sense of matching, to select non-matching lines.```

At the very bottom it outputted this:

![image](https://user-images.githubusercontent.com/41026969/72543442-b6e31d80-3853-11ea-9bc5-1f4ed5215c13.png)

Again, we see the root hash.

### Flags
user: ```/home/pi/Desktop/user.txt```

root: See above to get hash


### Sources
- [Info on port 53](https://www.grc.com/port_53.htm)
- [Info on DNSmasq](https://en.wikipedia.org/wiki/Dnsmasq)
- [Pi-hole wiki](https://en.wikipedia.org/wiki/Pi-hole)
- [SSH into Raspberry Pi](https://itsfoss.com/ssh-into-raspberry/)
- [df man page](https://linux.die.net/man/1/df)
- [grep man page](https://linux.die.net/man/1/grep)
- [Recovering text stack exchange question](https://unix.stackexchange.com/questions/2677/recovering-accidentally-deleted-files)
- [Mirai video writeup by IppSec](https://www.youtube.com/watch?v=SRmvRGUuuno)