---
layout: post
title: HackTheBox OpenAdmin Writeup
---

OpenAdmin is a box with an easy to medium level process of obtaining the user credentials, into which gaining the root shell is a matter of research and simple enumeration. The box features two users: Jimmy and Joanna that we had to get access into to eventually elevate to root. The main components of gaining unauthorized root access is as follows:

* Public facing of an OpenNetAdmin interface to the public. This also includes showing version number and having an outdated version being used.
* Plaintext credentials found in `/opt/ona/www/local/config/database_settings.inc.php`.
* Password reuse of user Jimmy.
* Plaintext index file showing functionality, a clear hash with encoding algorithm is also shown.
* Private RSA key is shared between users.
* User Joanna can do a sudo command without supplying a password.


Read the full writeup [here](https://securitynoodle.github.io/writeups/HTB-OpenAdmin.pdf).
