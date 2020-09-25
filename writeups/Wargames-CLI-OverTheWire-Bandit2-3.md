---
layout: page
title: Bandit Level 2 to Level 3
---

Getting the password from the last level, we can loggon to Bandit Level 2:

```
$ ssh bandit.labs.overthewire.org -p 2220 -l bandit2
```

Reading the challenge:
![2 to 3](https://user-images.githubusercontent.com/41026969/50000506-c5cc1200-ff67-11e8-8cdf-8cc54f40b676.png)

The recommended commands to read up are the same (refer to the Level 0 -> 1 writeup [here](https://github.com/BurntxNoodle/CTF/tree/master/OverTheWire%20-%20Bandit/Level%200%20-%3E%201) or google if necessary).

So according to the discription we know we will be dealing with a file name with saces, specifically called ```spaces in this filename``` 

Doing a quick ```ls -la``` reveal that there is indeed confirms that there is a file called ```spaces in this filename```

Googling on how to read/cat a filename we spaces we discover that there a few workarounds.

# Solution

There are a few ways to do this, I'll go over each one.

### First way

The first solution that I did was to simply start writing out the file name, and simply press the tab button as the linux
command line will simply fill out the rest. Specifically in the middle of typing out ```space``` I tabbed and linux
auto completed it for me.
```
bandit2@bandit:~$ cat spaces\ in\ this\ filename 
```

This will give us the password!
```
UmHadQclWmgdLOKQ3YNgjWxGoRMb5luK
```

### Second way

So using the tab button auto completed and pretty much revealed the second way of writing out the file name with spaces. We can 
insert a ```\``` before every space in the file name:

```
bandit2@bandit:~$ cat spaces\ in\ this\ filename 
```

Giving us the same password:
```
UmHadQclWmgdLOKQ3YNgjWxGoRMb5luK
```

### Third and final way

Since we know the final name, we can insert single quotation marks around the name without having the trouble of
having the insert a ```\``` after every space. It will look like this:
```
bandit2@bandit:~$ cat 'spaces in this filename'
```

This will yield us the same answer:
```
UmHadQclWmgdLOKQ3YNgjWxGoRMb5luK
```





