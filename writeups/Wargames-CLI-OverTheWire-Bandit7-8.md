---
layout: page
title: Bandit Level 7 to Level 8
---

First connecting to level 7 challenge:

```
$ ssh bandit.labs.overthewire.org -l bandit7 -p 2220
```

# Solution

So since it's next to word ```miliionth``` we can use the ```grep``` to find the word millionth and the strings around it.

We will have to use the pipe ```|``` character to pass it into grep as shown:

```
bandit7@bandit:~$ cat data.txt | grep millionth
```

This will give us the password for level 8: ```cvX2JJa4CFALtqS87jk27qwqGhBM9plV```