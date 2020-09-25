---
layout: page
title: Bandit Level 0 to Level 1
---

Note: You should still be logged in into Bandit0 from the previous challenge.

The challenge reads: 

![level 0 - 1](https://user-images.githubusercontent.com/41026969/49900055-784a8a80-fe2b-11e8-87dd-a317972197d5.png)

We can see it recommends that we read up on the following commands:

```
ls
cd
cat
file
du
find
```
### It's recommended to google or read man pages for these commands to view their whole argument list, syntax, etc.

Here is a basic summary of what each of the commands do:

```ls``` will list the directory contents. I usually like to add on ```-la``` to list all directory contents,
including hidden files, and it will give us the list in a long listed format, making it easier to read.

```cd``` stands for change directory. In later challenges we will see that there a multiple directories tha we have
to traverse into. We will be using the ```cd``` command in those situations. However, in this challenge we will
not be using cd.

```cat``` stands for concatenate. This command is used to print to standard output, usually to read contents
for a txt file.

```file``` is used for investigating the file type.

```du``` stands for disk usage. Though I haven't had to use this command in this wargame yet. According to the 
man page du will: ```summarize disk usage of each FILE, recursively for directories. ``` 

```find``` will search for files in a directory hierarchy. We will see in later challenges we can add certain
parameters to add specific restrictions.

# Solution:

First we will ```ls -la```. To view all the contents of the file:

```
bandit0@bandit:~$ ls -la
```

This will show us the files:

![ls -la](https://user-images.githubusercontent.com/41026969/49900510-c4e29580-fe2c-11e8-8022-d21383b5c4dc.png)

We see that there is a file called ```readme```. Looking into what kind of file it is using the ```file``` command:

```
bandit0@bandit:~$ file readme 
readme: ASCII text
```

To read a ASCII text (essentially a txt file) we use the ```cat``` command:
```
bandit0@bandit:~$ cat readme 
```
This will give us the password for the next level!
Password: ```boJ9jbbUNNfktd78OOpsqOltutMc3MY1```