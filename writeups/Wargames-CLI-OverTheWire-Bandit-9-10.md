---
layout: page
title: Bandit Level 9 To Level 10
---

First connect to Level 9: ```ssh bandit.labs.overthewire.org -p 2220 -l bandit9``` and enter the password from the previous challenge: ```UsvVyFSfZZWbi6wgC7dAFyFuR6jQQUhR```

This is what the challenge says:

![bandit9](https://user-images.githubusercontent.com/41026969/51123993-3b3b3480-17eb-11e9-9d8d-5df271cfc386.png)

Logging into the challenge we see a singluar file name: ```data.txt```. Even though it does end it a ```.txt```, doing a ```file data.txt``` check says that it contains data (sometimes opcodes, or encoded data) and thus it's not human readable, you can check this by doing ```cat data.txt``` and seeing that it spits out a bunch of garbage. 

One way we can look at human readable characters in an encoded file is by using the ```strings``` command, according to the ```man``` page: ```strings - print the strings of printable characters in files.```

Combining the above information, with the challenge description (particularly where it says ```beginning with several '=' characters)``` we know we can first find the human readable strings and then filter all the results to only show strings with '='. So we use ```strings data.txt``` then pass the output into ```grep '==='``` using the pipe (```|```) character.

This is how the result looks like:

![screenshot from 2019-01-14 11-59-00](https://user-images.githubusercontent.com/41026969/51127590-d20bef00-17f3-11e9-8b56-a6096c6da1c3.png)

Thus we get the password for the next level!
```truKLdjsbJ5g7yyJ2X2R0o3a5HQJFuLk```