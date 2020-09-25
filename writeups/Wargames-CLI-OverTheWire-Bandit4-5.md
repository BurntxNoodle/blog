---
layout: page
title: Bandit Level 4 to Level 5
---

Using the password obtained from the last challenge we can log into bandit 4:
```
$ ssh bandit.labs.overthewire.org -p 2220 -l bandit4
```
This is what the challenge says:

![4 to 5](https://user-images.githubusercontent.com/41026969/50039427-53f3d700-0000-11e9-9de4-8635342a0d48.png)

So we're going to have to google or find out ways to find a human readable file.

First let's change to the ```inhere``` directory:

```
bandit4@bandit:~$ cd inhere/
```

# Solution

Doing a ```ls -la``` check yields the following result:

![list](https://user-images.githubusercontent.com/41026969/50039440-b1882380-0000-11e9-9d10-8baea7a894b6.png)  

So we get a result of 10 different files (with a dashed filename: good thing we know a workaround from the previous  
challenges).

My first solution is to use the ```*``` character. This character in Linux will return results that matched with
the starting characters preceding the ```*```. 

For example, instead of doing a ```file``` check for every single file, we can use the ```*``` character to our advantage. We
see that each of the files begin with ```-file```. Thus we can use the ```*``` character correcectly to return to us all the
```file``` checks for all of them as shown:

```
bandit4@bandit:~/inhere$ file ./-file*
```

This will return:

![asterisk](https://user-images.githubusercontent.com/41026969/50039498-9bc72e00-0001-11e9-8bad-10fd2f63fd56.png)


We notice that ```./-file07``` is of type ASCII. Let's cat it:
```
bandit4@bandit:~/inhere$ cat ./-file07
```

This will spit out the password:
```koReBOKuIDDepwhWk7jZC0RTdopnAYKh```

The challenge is complete!

