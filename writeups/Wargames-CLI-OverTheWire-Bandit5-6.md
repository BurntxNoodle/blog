---
layout: page
title: Bandit Level 5 to Level 6
---

To connect to Level 5:

```
$ ssh bandit.labs.overthewire.org -p 2220 -l bandit5
```

And now we read the challenge:

![5 to 6](https://user-images.githubusercontent.com/41026969/50047805-24d67780-008b-11e9-86ff-91f1008bd826.png)

We see that we will have to find a file with certain specifications. Doing an ```ls -la``` check in the ```inhere``` directory
reveals that there are indeed a lot of files. The ```inhere``` directory contains 20 other directories that all have a few files. 

Doing individual checks for each of the files would be tedious. Thus we need to utilize the ```find``` command.

Reading the man page on how to find a file of a specific size, we see we can add on ```-size n[cwbkMG] ``` where ```n``` is the size and ```c, w, b, k, M,``` or ```G``` (stands for byte, two byte word, kilobyte, megabyte, and gigabyte, respectively.

Also find a human readable file, we can tack on ```-readable```

Last, to filter out executables we see from the man page that there is an executable check you can do with ```-executable```


# Solution

To combine all of these restrictions in one command we do:
```
bandit5@bandit:~/inhere$ find -readable ! -executable -size 1033c
```

This will give us the location of the file: ```bandit5@bandit:~/inhere$ find -readable ! -executable -size 1033c```

Using the ```cat``` command: 

```
bandit5@bandit:~/inhere$ cat ./maybehere07/.file2
```

This will give us the answer for level 6!
```
DXjZPULLxYr17uwoI01bNLQbtFemEgo7
```
(note you do not include the spaces when typing in the password for access to the next level)