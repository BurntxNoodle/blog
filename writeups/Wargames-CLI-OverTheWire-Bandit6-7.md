---
layout: page
title: Bandit Level 6 to Level 7
---

First let's connec to level 6:  
```
$ ssh bandit.labs.overthewire.org -l bandit6 -p 2220
```
The challenge:

![6 to 7](https://user-images.githubusercontent.com/41026969/50057925-46913680-013f-11e9-85b9-c601eb2f170c.png)  

So we see that we will have to use the ```find``` command again, although this time there are more restrictions
and things to search for since according to the challenge, it is somewhere on this server. This means 
we will have to search starting from the root directory.

Reading up on the man page of ```find``` we see that we can add on certain thigns to search files owned by a specific
user and by a specific group:

To use ```find``` to return a file that's owned by a specific group we can add on ```-group gname``` (gname being the groupname
associated with the file).

To use ```find``` to return a file that's owned by a specific user we can add on ```-user uname``` (uname being the user name)

Using a combination of these will yield us the answer:

# Solution

So when we automatically ssh into a system, we start in the home directory. However, since the challenge description
says it's ```somewhere on this server``` we must start the ```find``` in the root directory. Also, adding in the
filters previously mentioned in this writeup:

```
bandit6@bandit:~$ find / -user bandit7 -group bandit6 -size 33c
```
This will produce us a pretty long output with multiple files being 33 bytes, owned by bandit7:

![long find](https://user-images.githubusercontent.com/41026969/50058129-1b5c1680-0142-11e9-915d-cdb3778204b6.png)

We see that all but one of the files say ```Permission denied``` and the one file is: ```/var/lib/dpkg/info/bandit7.password```

Using ```cat```: 

```
bandit6@bandit:~$ cat /var/lib/dpkg/info/bandit7.password
```

This will give us the password: ```HKBPTKQnIay4Fw76bEy8PVxKEDQRKTzs```  

Note: we can filter out those search results from ```Permission denied``` by adding on ```2>/dev/null``` at the end of our
find command. This will remove stderr (standard error) outputs as seen here:

![filter](https://user-images.githubusercontent.com/41026969/50058259-af7aad80-0143-11e9-996e-a385e362364f.png)