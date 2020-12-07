---
layout: post
title: HackTheBox Celestial Writeup
---

This is a medium level HackTheBox machine that involves around a deserialization exploit for Node.js that lead to getting an initial access point on the machine. Afterwards, gaining a root shell was very simplistic due to misconfigurations. The main components of gaining unauthorized root access is as follows:

- Burpe Suite was utilized to receive a cookie which was then discovered to be editable before forwarding the web request.
- Since it was possible to edit the cookie (essentially user input) that passes information to a function that utilizes the eval() function.
- A custom, serialized payload in Node.js had to be created. Once it has been created, encoding it to the cookie format and forwarding the web request was done to achieve an initial foothold.
- Afterwards, a cron job that runs every 5 minutes was found running a python script as root. Even though the script was running as root, it was owned by the user that was compromised by the initial foothold.
- Edit/upload a new script that generates a reverse shell as root.

Read the full writeup on [my medium blog post!](https://securitynoodle.medium.com/hackthebox-celestial-writeup-1a2022427c50)