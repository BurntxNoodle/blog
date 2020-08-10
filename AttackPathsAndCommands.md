---
layout: page
title: Attack Paths and Commands
---

# 🚧 Project in progress 🚧

## What is this page?

This page will contain a high-level overview of general attack paths/vectors that I've encountered from activities such as HackTheBox and VulnHub. Additionally, this page will have a checklist of commands that maps out to each of its respective attack path. For ease of use and organization, each of the attack paths and their commands are organized based on the [MITRE ATT&CK](https://attack.mitre.org/) framework.

## Reconnaissance and Enumeration

* [General nmap scan]()
* [Nmap scan for all ports]()
* [General zenmap scan]()
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

##### General nmap scan
`nmap -sC -sV -oA resolute 10.10.10.169`

![image](https://user-images.githubusercontent.com/41026969/89837667-702e7b00-db37-11ea-9c5e-fb19e0846ad4.png)

* `nmap`: The tool being used
* `-sC`: Use default scripts
* `-sV`: Probe the open ports found to find and determine service/version information
* `-oA`: Output all formats (`<NAME>.nmap` is the text based output)
* `resolute`: Base name for the output files. In this case, it will write results to a file named `resolute.nmap` (with two additional formats)
* `10.10.10.169`: IP address that will be scanned

##### Nmap scan for all ports

##### General zenmap scan

##### Using gobuster to brute-force URIs

##### Using dirb to brute-force URIs

##### Using dirbuster to brute-force URIs

##### Using wfuzz to fuzz potential files

##### Using wfuzz to fuzz subdomains
