---
layout: page
title: Attack Paths and Commands
---
Last updated: 9/11/2020

# 🚧 Project in progress 🚧

## What is this page?

This page will contain a high-level overview of general attack paths, vectors and commands that I've utilized from activities such as HackTheBox and VulnHub. For organization and ease of use, each of the attack paths and their commands are organized based on the [MITRE ATT&CK](https://attack.mitre.org/) framework.

## Reconnaissance and Enumeration

* [General nmap scan](https://securitynoodle.github.io/RedTeam/AttackPathsAndCommands/#general-nmap-scan)
* [Nmap scan for all ports](https://securitynoodle.github.io/RedTeam/AttackPathsAndCommands/#nmap-scan-for-all-ports)
* [Using gobuster to brute-force URIs](https://securitynoodle.github.io/RedTeam/AttackPathsAndCommands/#using-gobuster-to-brute-force-uris)
* [Using dirb to brute-force URIs](https://securitynoodle.github.io/RedTeam/AttackPathsAndCommands/#using-dirb-to-brute-force-uris)
* [Using dirbuster to brute-force URIs](https://securitynoodle.github.io/RedTeam/AttackPathsAndCommands/#using-dirbuster-to-brute-force-uris)
* [Using wfuzz to fuzz potential files or URIs](https://securitynoodle.github.io/RedTeam/AttackPathsAndCommands/#using-wfuzz-to-fuzz-potential-files-or-uris)
* [Using wfuzz to find subdomains](https://securitynoodle.github.io/RedTeam/AttackPathsAndCommands/#using-wfuzz-to-find-subdomains)
* [Using sublist3r to find subdomains](https://securitynoodle.github.io/RedTeam/AttackPathsAndCommands/#using-sublist3r-to-find-subdomains)

## Initial Access, Execution

* Check for SQL injection
* [Windows] Is SMB exposed (eternal blue)
* [Windows] Is IIS version < (insert version here)


## Privilege Escalation

* [[Linux] Checking sudo commands that do not require password](https://securitynoodle.github.io/RedTeam/AttackPathsAndCommands/#checking-sudo-commands-that-do-not-require-password)
* [Windows] Is savecred allowed
* [Windows] Is DNSAdmin 

## Discovery

* [[Linux] Checking ports that are listening](https://securitynoodle.github.io/RedTeam/AttackPathsAndCommands/#checking-ports-that-are-listening)
* [Linux] Checking for files owned/accessible
* [[Windows] Checking what groups a user is in](https://securitynoodle.github.io/RedTeam/AttackPathsAndCommands/#checking-what-groups-a-user-is-in)
* [[Windows] Checking what users are in an Active Directory group](https://securitynoodle.github.io/RedTeam/AttackPathsAndCommands/#checking-what-users-are-in-an-active-directory-group)

## Lateral Movement

* [Windows] Using Mimikatz
* [Windows] Using BloodHound

## Exfiltration and File Transferring

* [Downloading files from FTP](https://securitynoodle.github.io/RedTeam/AttackPathsAndCommands/#downloading-files-from-ftp)

## Post Exploitation 

<p align="center">
  <img src="https://user-images.githubusercontent.com/41026969/89838415-2cd50c00-db39-11ea-824b-8ef86b869974.png" />
</p>

# Reconnaissance and Enumeration Commands

### General nmap scan
`nmap -sC -sV -oA <name> <address>`

Example: `nmap -sC -sV -oA resolute 10.10.10.169`

![image](https://user-images.githubusercontent.com/41026969/89837667-702e7b00-db37-11ea-9c5e-fb19e0846ad4.png)

* `nmap`: The tool being used
* `-sC`: Use default scripts
* `-sV`: Probe the open ports found to find and determine service/version information
* `-oA resolute`: Output all formats to `resolute` (`<NAME>.nmap` is the text based output) where resolute is the base name for the output files; in this case, it will write results to a file named `resolute.nmap` (with two additional formats)
* `10.10.10.169`: IP address that will be scanned

### Nmap scan for all ports
`nmap -p- -sC -sV -oA <name> <address>`

Example: `nmap -p- -sC -sV -oA allports 10.10.10.169`

![image](https://user-images.githubusercontent.com/41026969/89954677-8ef84480-dbff-11ea-9dc1-e16ba96e8a7b.png)

* `nmap`: The tool being used
* `-p-`: Scan all 65535 ports
* `-sC`: Use default scripts
* `-sV`: Probe the open ports found to find and determine service/version information
* `-oA allports`: Output all formats to `allports` (`<NAME>.nmap` is the text based output) where `allports` is the base name for the output files; in this case, it will write results to a file named `allports.nmap` (with two additional formats)
* `10.10.10.169`: IP address that will be scanned


### Using gobuster to brute-force URIs
`gobuster dir -u <address> -w <wordlist> -o <name> -t <threads>`

Example: `gobuster dir -u http://10.10.10.75/nibbleblog/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -o gobuster_results.txt -t 100`

![image](https://user-images.githubusercontent.com/41026969/74098356-3b3c5f80-4ae5-11ea-85ae-68c200a0e0c8.png)

* `gobuster`: The tool being used
* `dir`: directory mode
* `-u http://10.10.10.75/nibbleblog/`: `-u` specifying the base URL to check which is `http://10.10.10.75/nibbleblog/` in this example
* `-w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt`: `-w` specifying the wordlist to use which is `/usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt` in this example
* `-o gobuster_results.txt`: Output results to the specified file, in this case `gobuster_results.txt`
* `-t 100`: Number of threads to use which in this example is `100`

### Using dirb to brute-force URIs
`dirb <url> <wordlist>`

Example: `dirb http://10.10.10.75/nibbleblog /usr/share/wordlists/dirb/common.txt`

![image](https://user-images.githubusercontent.com/41026969/91897178-e53d2e00-ec67-11ea-8857-00595b757817.png)

* `dirb`: The tool being used
* `http://10.10.10.75/nibbleblog`: The base URL to check against
* `/usr/share/wordlists/dirb/common.txt`: The wordlist to use

##### Note:
This tool slow as heck. Use gobuster or dirbuster.

### Using dirbuster to brute-force URIs
Dirbuster is the GUI version of dirb, to bring it up by either searching for `dirbuster` in the kali linux start menu or by typing `dirbuster` in the terminal. 

The fields are pretty self-explanatory, here's an example of the fields being filled out for the same example as above:

![image](https://user-images.githubusercontent.com/41026969/91897439-4bc24c00-ec68-11ea-8b30-15640ab5ccb2.png)

After clicking the `Start` button on the bottom left, I like to go to the `Results - Tree View` tab to see what dirbuster finds.

![image](https://user-images.githubusercontent.com/41026969/91897710-b1163d00-ec68-11ea-8876-9ef7c3df7999.png)

### Using wfuzz to fuzz potential files or URIs
`wfuzz --hc 404 -Z -w <wordlist> -u <domain>/FUZZ/<file name>`

##### Context: 
In the HackTheBox challenge [Obscurity](https://securitynoodle.github.io/RedTeam/HackTheBox/#hackthebox-obscurity), I am given that there is a file named `SuperSecureServer.py` in a secret directory on the server. So I know that there HAS to be some URL that exists that looks like `http://10.10.10.168:8080/SOMETHING/SuperSecretServer.py` where `SOMETHING` is the directory that we need to find out. To find out that directory, we use wfuzz!

Example `wfuzz --hc 404 -Z -w /usr/share/wordlists/dirbuster/directory-list-2.3-small.txt
-u http://10.10.10.168:8080/FUZZ/SuperSecureServer.py`

![image](https://user-images.githubusercontent.com/41026969/91778568-e6ffe680-ebc0-11ea-92cc-981423b27e31.png)

* `wfuzz`: The tool being used
* `--hc 404`: `--hc` hides the code specified, in this case `404` to not show 404 pages
* `-Z`: Scan mode - connection errors will be ignored
* `-w /usr/share/wordlists/dirbuster/directory-list-2.3-small.txt`: `-w` specifies the wordlist to use, in this case `/usr/share/wordlists/dirbuster/directory-list-2.3-small.txt`
* `-u http://10.10.10.168:8080/FUZZ/SuperSecureServer.py`: `-u` specifies the URL to use, in this case `http://10.10.10.168:8080/FUZZ/SuperSecureServer.py`. Make sure you use the `FUZZ` keyword which symbolizes what you want to check/replace with the wordlist.

### Using wfuzz to find subdomains
`wfuzz --sc 200 -w <wordlist> -H "Host: FUZZ.<domain>" -u <domain>`

Example: `wfuzz --sc 200 -w /usr/share/wordlists/dirb/common.txt -H "Host: FUZZ.sneakycorp.htb" -u http://sneakycorp.htb`

![image](https://user-images.githubusercontent.com/41026969/91776965-9be3d480-ebbc-11ea-8427-6e87829a32dc.png)

* `wfuzz`: The tool being used
* `--sc 200`: Only show responses with the specified code, in this example `200` (shows valid pages)
* `-w /usr/share/wordlists/dirb/common.txt`: `-w` specifies the wordlist to use, in this case `/usr/share/wordlists/dirb/common.txt`
* `-H "Host: FUZZ.sneakycorp.htb"`: `-H` specifies what to add in the header in the request to the website, in this example `"Host: FUZZ.sneakycorp.htb"`
* `-u http://sneakycorp.htb`: `-u` is the URL to use, in this case `http://sneakycorp.htb`

### Using sublist3r to find subdomains
##### Note: To install:
* `sudo git clone https://github.com/aboul3la/Sublist3r /opt/sublist3r`

`python3 sublist3r.py -d <domain>`

Example: `python3 sublist3r.py -d cyberspacekittens.com`

![image](https://user-images.githubusercontent.com/41026969/91775272-adc37880-ebb8-11ea-8442-2ccef74c7909.png)

* `python3 sublist3r.py`: This program is a python3 script, so use python3
* `-d cyberspacekittens.com`: `-d` specifies the domain to check, in this example `cyberspacekittens.com`

<p align="center">
  <img src="https://user-images.githubusercontent.com/41026969/89838415-2cd50c00-db39-11ea-824b-8ef86b869974.png" />
</p>

# Privilege Escalation Commands

### Checking sudo commands that do not require password
`sudo -l`

Example:
![image](https://user-images.githubusercontent.com/41026969/92505030-76b12080-f1d1-11ea-96c3-c1a098649f32.png)

In this example, you can see the user can run the command `sudo /bin/nano /opt/priv` as sudo without supplying a password.

Nano can spawn a shell, and since it's running under sudo, it will spawn a shell as sudo [source here](https://gtfobins.github.io/gtfobins/nano/#shell).

```
nano
^R^X
reset; sh 1>&0 2>&0
```
<p align="center">
  <img src="https://user-images.githubusercontent.com/41026969/89838415-2cd50c00-db39-11ea-824b-8ef86b869974.png" />
</p>

# Discovery

### Checking ports that are listening
`netstat -tulp`
* Utilizes the netstat command to look for tcp (the `t` in the command) and udp (the `u` in the command) ports that are listening (`l`) and prints output with program names (the `p`)

### Checking what groups a user is in 
`net user <user name> /domain`

Example: `net user ryan /domain`
* Gets information on the user `ryan` on the primary domain. The groups will be listed under `Global Group Memberships` and/or `Local Group Memberships`.

### Checking what users are in an Active Directory group
`Get-ADGroupMember <AD group> | select name`

Example: `Get-ADGroupMember Contractors | select name`
* Gets all members of the `Contractors` AD group, then from that list only output the names.

<p align="center">
  <img src="https://user-images.githubusercontent.com/41026969/89838415-2cd50c00-db39-11ea-824b-8ef86b869974.png" />
</p>

# Exfiltration and File Transferring Commands

### Downloading files from FTP
##### Note: it's important to distinguish if you need to utilize FTP's ASCII mode or Binary mode, examples of file types that should be used for each mode is listed below. When initially connected to FTP, ASCII mode is the default mode.

`get "<file name>"`

Example: `get "Access Control.zip"`

To change mode, simply type `binary` in the FTP shell. So if you wanted to get a file in binary mode:
* `> binary`
* `> get "backup.mdb"`

##### Examples of file types to use ASCII Mode
* .txt
* .asp
* .html
* .php
* (Generally text-based files, such as source code, web pages, etc.)

##### Examples of file types to use Binary Mode
* .jpg
* .gif
* .wave
* .mp3
* .mp4
* .mov
* .zip
* .tar
* .mdb
* .exe
* .doc
* .pdf
* (Generally image files, sound files, video files, archive files, etc.)

<p align="center">
  <img src="https://user-images.githubusercontent.com/41026969/89838415-2cd50c00-db39-11ea-824b-8ef86b869974.png" />
</p>