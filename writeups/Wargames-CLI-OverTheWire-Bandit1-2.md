---
layout: page
title: Bandit Level 1 to Level 2
---

You should be able to exit out of the ssh by just typing ```quit```

Now that you have the password for the next level, you can connect to the next level, except
that you have to change the username to specification ```-l banditX``` where x is the level you are
accessing. 

Reading the challenge:

![screenshot from 2018-12-14 00-15-06](https://user-images.githubusercontent.com/41026969/49984282-6b19c280-ff35-11e8-84ce-70be47325c2a.png)

The helpful commands list is the same from Level 0 -> 1. You can refer to it [here](https://github.com/BurntxNoodle/CTF/tree/master/OverTheWire%20-%20Bandit/Level%200%20-%3E%201) or simply google them.

To connect to bandit level 1:
```
$ ssh bandit.labs.overthewire.org -p 2220 -l bandit1
```

Googling on how to cat a dashed filename, we discover that there a couple work arrounds.

# Solution

After we connect into the first level of the wargame, we do a quick ```ls -la``` to view all files:
```
bandit1@bandit:~$ ls -la
```

We see that there is a filename called ```-```

One of the workarounds is that before the ```-``` in the filename when we cat it, we add ```./``` before. Resulting in:
```
bandit1@bandit:~$ cat ./- 
```

This will spit out the password for level 2: ```CV1DtqXWVFXTvM2F0k09SHz0YwRINYA9```

A different workaround is that we can go into the directory in which the file is. We know that we are in level 1 and
we can look through and access files from the home directory. It is worth noting that after ```/home``` I simply tabbed and saw
all the bandit levels appear but we only have access to the level we are currently on. Here's how the command looks like:

```
bandit1@bandit:~$ cat /home/bandit1/-
```

This also yields us the same password: ```CV1DtqXWVFXTvM2F0k09SHz0YwRINYA9```