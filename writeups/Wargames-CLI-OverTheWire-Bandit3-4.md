---
layout: page
title: Bandit Level 3 to Level 4
---

Using the password we got from the previous level, we can access Bandit 3:
```
$ ssh bandit.labs.overthewire.org -p 2220 -l bandit3
```

Reading the challenge:

![3 to 4](https://user-images.githubusercontent.com/41026969/50001204-1f354080-ff6a-11e8-978a-917958241c8e.png)

The recommened commands are the same as the past few. You can read up on them [here](https://github.com/BurntxNoodle/CTF/tree/master/OverTheWire%20-%20Bandit/Level%200%20-%3E%201) or google/read the man pages.

Reading up on hidden file names and reading the ```ls``` man page, we know we can add on ```-a``` to list all the 
files, including hidden files. *this shouldn't be new to us as I've been doing ```ls -la``` for the past challenegs*

Linux hidden files are denoted by the ```.``` before their filename. 

Changing the directories and using cat to read the file name should be the same:

# Solution

Doing a quick ```ls -la``` reveals that there is a hidden file called ```.inhere``` 

Doing a ```file``` check reveals that it is a directory.

We change directory into it as normal (we do not need to include the ```.```)
```
bandit3@bandit:~$ cd inhere/
```

Since we are in a new directory, we do another ```ls -la``` to view the contents:
```
bandit3@bandit:~/inhere$ ls -la
```

We see that there is a file called ```.hidden``` and doing a ```file``` check reveals that it contains ASCII text.

To cat this, we include the ```.``` in the file name as shown:
```
bandit3@bandit:~/inhere$ cat .hidden 
```

This will spit out the password: ```pIwrPrtPN36QITSp3EQaw936yaFoFgAB```




