First let's read the challenge description:

![bof](https://user-images.githubusercontent.com/41026969/50720410-e2e19580-107a-11e9-8c4e-760fdb4f0d87.PnG)

So we can tell we'll be working on a buffer overflow challenge. Before we begin, there's a few things you need to understand before getting into how buffer overflow works:

1. x86 Assembly [Good Video Here](https://www.youtube.com/watch?v=75gBFiFtAb8)
2. Function Prologue/Epilogue [Wiki Here](https://en.wikipedia.org/wiki/Function_prologue)
3. General knowledge of stacks (combination of both of the above)

So for this challenge, we are given the source code, let's open that first (source file also provided above):

![bof](https://user-images.githubusercontent.com/41026969/50780253-78924600-1270-11e9-85e0-b19fc6ca36d8.png)

So we see that this function uses ```gets()``` which is extremely vulnerable since it doesn't check if what you inputted goes past the buffer. To explain this better: in C programs (like this one) the variable is only supposed to contain 32 chars, but ```gets()``` does not check if what you inputted was more than 32 characters, and it will continue writing whatever you input into memory.

Unfortunately, since ```overflowme``` is in a local function, it's on some negative offset. Also, the argument passed (```0xdeadbeef1```) is a function parameter so it also starts on some negative offsets. This means that we can't really figure out (yet) how much bytes to feed.

There are a couple workarounds to find the exact number of bytes we need to give ```gets()``` :

### The first workaround
So first we can fire up gdb debugger. This allows us to view registers, view values stored in those registers, memory addresses, disassembly of function and more.

First, to open gdb and run the ```bof``` executable, we just type in ```gdb bof```. Then I just run through the program so it can examine everything as seen below:

![bof](https://user-images.githubusercontent.com/41026969/50835773-9d94c080-1325-11e9-9eab-8df30d59d310.png)

```r``` is just the command to run the program and the rest runs like normal. 

Next, since we are given the source code we know that the function we want to look at is named ```func```. We can disassemble the function by doing ```disass func``` as shown below:

![bof](https://user-images.githubusercontent.com/41026969/50836056-44795c80-1326-11e9-99e0-4301b56a7cb6.png)

First, we see at address ```0x5655564f``` we see that the assembly calling the ```gets()``` function (in gdb it doesn't say what the function is, but you can also open up IDA's disassembly and the instruction order will be the same except you'll see it calls the ```gets()``` function. Next we can see that the next instruction is a compare instruction (at address ```0x56555654```). It compares ```0xcafebabe``` to a value stored at ```ebp + 0x8```. 

So theoretically, we can set a breakpoint at the ```cmpl``` function, thus it will pause the program after we input something from the ```gets()``` command. 

So we can input a bunch of stuff and try to find how many characters it takes until we overwrite the address stored in ```ebp + 0x8``` as shown here:

![bof](https://user-images.githubusercontent.com/41026969/50838215-7e992d00-132b-11e9-9bf7-a7b63b78617d.png)

So we do ```b *[address]``` sets a break point at the ```address``` we set it to, in this case we set the breakpoint right before the program compares the addresses. Then we rerun the program by typing ```r```.  When we get asked for the input, to get the exact amount of bytes we need to give it, so we do each letter 4 at a time because this program is 32 bits, which is 4 byte addresses. 

We then hit the breakpoint, in which we can examine what's stored into ```$ebp + 0x8``` by doing x/wx [(command explanation here)[http://visualgdb.com/gdbreference/commands/x]. Thus we type the command: ```x/wx $ebp + 0x8```:

![bof](https://user-images.githubusercontent.com/41026969/50840756-3a109000-1331-11e9-97ff-06fd1a88bfb2.png)

So we see that ```0x4e4e4e4e``` was overwritten into the address. Checking what 4e is, we see it is equal to the letter```N```. Therefore, we can just copy and the letters A through M, and write in the address. I'm learning the pwntools library in python so I decided to write a simple python program (or you can manually do it yourself):
```python
from pwn import *

session = remote("pwnable.kr", 9000)

payload = "AAAABBBBCCCCDDDDEEEEFFFFGGGGHHHHIIIIJJJJKKKKLLLLMMMM" + "\xbe\xba\xfe\xca"

session.sendline(payload)

session.interactive()
```
After we overwrite the buffer, the ```session.interactive()``` allows us to get a shell. Doing a quick ```whoami``` check reveals we are bof and that we can now read the flag. ```cat flag``` gives us the flag: ```daddy, I just pwned a buFFer :)```

