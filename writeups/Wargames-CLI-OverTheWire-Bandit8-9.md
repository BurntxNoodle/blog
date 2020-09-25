---
layout: page
title: Bandit Level 8 to Level 9
---

First we'll connect to Level 8 ```ssh bandit.labs.overthewire.org -p 2220 -l bandit8``` and enter the password from the last challenge: ```cvX2JJa4CFALtqS87jk27qwqGhBM9plV```

This is what the challenge says:

![level 8](https://user-images.githubusercontent.com/41026969/50865964-0a37ab80-1376-11e9-9151-861ef168f447.png)

So from the description, we know that we have to find a unique line of code. This means will be using the ```uniq``` command to find out the unique lines. Reading the manpage, we see that we can only print unique lines by doing ```uniq -u```. 

So when do that, it still prints out all the lines. This is because ```uniq``` only compares line by line. So first we need to sort ```data.txt``` using sort, then pass it into the ```uniq -u``` command. Thus our command becomes ```sort data.txt | uniq -u```.

This yields us the result:

![level 8](https://user-images.githubusercontent.com/41026969/50866440-f2612700-1377-11e9-8b2c-7eaf9e62cd33.png)

The password: ```UsvVyFSfZZWbi6wgC7dAFyFuR6jQQUhR```