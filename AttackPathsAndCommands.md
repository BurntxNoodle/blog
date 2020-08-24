---
layout: page
title: Attack Paths and Commands
---

# 🚧 Project in progress 🚧

## What is this page?

This page will contain a high-level overview of general attack paths/vectors that I've encountered from activities such as HackTheBox and VulnHub. Additionally, this page will have a checklist of commands that maps out to each of its respective attack path. For ease of use and organization, each of the attack paths and their commands are organized based on the [MITRE ATT&CK](https://attack.mitre.org/) framework.

## Reconnaissance and Enumeration

* [General nmap scan](https://burntxnoodle.github.io/AttackPathsAndCommands/#general-nmap-scan)
* [Nmap scan for all ports](https://burntxnoodle.github.io/AttackPathsAndCommands/#nmap-scan-for-all-ports)
* [Using gobuster to brute-force URIs]()
* [Using dirb to brute-force URIs]()
* [Using dirbuster to brute-force URIs]()
* [Using wfuzz to fuzz potential files]()
* [Using wfuzz to fuzz subdomains]()

## Initial Access, Execution, and File Transfers

## Privilege Escalation

## Discovery

## Lateral Movement

## Exfiltration

## Post Exploitation 

<p align="center">
  <img src="https://user-images.githubusercontent.com/41026969/89838415-2cd50c00-db39-11ea-824b-8ef86b869974.png" />
</p>

### General nmap scan
`nmap -sC -sV -oA resolute 10.10.10.169`

![image](https://user-images.githubusercontent.com/41026969/89837667-702e7b00-db37-11ea-9c5e-fb19e0846ad4.png)

* `nmap`: The tool being used
* `-sC`: Use default scripts
* `-sV`: Probe the open ports found to find and determine service/version information
* `-oA`: Output all formats (`<NAME>.nmap` is the text based output)
* `resolute`: Base name for the output files; in this case, it will write results to a file named `resolute.nmap` (with two additional formats)
* `10.10.10.169`: IP address that will be scanned

### Nmap scan for all ports
`nmap -p- -sC -sV -oA allports 10.10.10.169`

![image](https://user-images.githubusercontent.com/41026969/89954677-8ef84480-dbff-11ea-9dc1-e16ba96e8a7b.png)

* `nmap`: The tool being used
* `-p-`: Scan all 65535 ports
* `-sC`: Use default scripts
* `-sV`: Probe the open ports found to find and determine service/version information
* `-oA`: Output all formats (`<NAME>.nmap` is the text based output)
* `allports`: Base name for the output files; in this case, it will write results to a file named `allports.nmap` (with two additional formats)
* `10.10.10.169`: IP address that will be scanned


### Using gobuster to brute-force URIs
`gobuster dir -u http://10.10.10.75/nibbleblog/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -o gobuster_results.txt -t 100`

![image](https://user-images.githubusercontent.com/41026969/74098356-3b3c5f80-4ae5-11ea-85ae-68c200a0e0c8.png)

* `gobuster`: The tool being used
* `dir`: directory mode
* `-u`: URL flag
* `http://10.10.10.75/nibbleblog/`: The base URL to check (from HackTheBox, writeup linked below)
* `-w`: Wordlist flag
* `/usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt`: Wordlist to use
* `-o`: Output results flag
* `gobuster_results.txt`: Name of file to output results to
* `-t`: Number of threads to use
* `100`: The number of threads this gobuster command will run

### Using dirb to brute-force URIs

### Using dirbuster to brute-force URIs

### Using wfuzz to fuzz potential files

### Using wfuzz to fuzz subdomains
