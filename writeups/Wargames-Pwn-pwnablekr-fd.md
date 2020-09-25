# fd writeup

First let's take a look at the challenge description says:

![fd](https://user-images.githubusercontent.com/41026969/50621778-35ce1800-0ed6-11e9-9ce3-e36e9487ec01.PNG)

So we know from the challenge description that fd also stands for file descriptor (more on this later).

So we open up a terminal and ssh into the address and enter the password ```guest```

First we do a quick ```ls -la``` to check all the files in the system. Doing so will reveal 3 files: an executable named ```fd```, a c source filed named ```fd.c``` and some text named ```flag```. We see that we do NOT have permission to read the flag text. (If you do not know how/need a refresher on how to read the linux permissions, here's a good [link to learn it](https://www.pluralsight.com/blog/it-ops/linux-file-permissions))

I like to play around with the executable before first inspecting it. Doing ```./fd``` reveals that we need to also pass in an argument, and trying out any integer/string will spit out ```learn about Linux file IO```...

Let's look at the source code:

![fd](https://user-images.githubusercontent.com/41026969/50622005-0c15f080-0ed8-11e9-93e2-4edf8681c6e9.PNG)

So looking at the code:

1) It takes whatever we pass in as an argument and subtracts it by ```0x1234```
2) Then we encounter a ```read()``` call. Doing a quick google search of read() in c, we figure out the syntax of it: ```read(int fildes, void *buf, size_t nbyte);```. So this is what the source code in the system says: ```len = read(fd, buf, 32);``` The ```fd``` is the result from the above, and the rest means we'll read in 32 bytes into the buffer.
3) It then compares the buffer to ```LETMEWIN\n```. If it passes then we get the flag, if not then it exits.

So looking up file descriptors in linux, we see that

0 = Standard Input
1 = Standard Output
2 = Standard Error

We want the buffer to read into standard input since we want to pass in input, and Standard Input is file descriptor 0. So working backwards, we need the fd to equal 0. But in the beginning of the function we see that the ```fd``` variable is subtracted by 0x1234... The Windows calculator app actually has different modes, and one of which is a ```Programmer``` mode. Click the 3 lines on the top left, then click programmer. After which, we can click the ```HEX``` so that the calculator knows we're inputting hex numbers. I simply put ```0 + 1234``` and here's what we get:

![fd](https://user-images.githubusercontent.com/41026969/50622643-c0b21100-0edc-11e9-939e-4ab26676faa9.PNG)

We see that we can the decimal value equivalent that we'd have to pass into the program (there's no way the computer can tell we're passing a hex value, it'll just assume it's decimal), which is ```4660```.

Doing ```./fd 4660``` we see the program is reading for a next input to read into the buffer. Looking at the code, we should type in ```LETMEWIN\n``` (we don't actually type the \n, that just means new line which gets inputted when we click enter after). 

Doing so gets us the flag!!
```mommy! I think I know what a file descriptor is!!```
