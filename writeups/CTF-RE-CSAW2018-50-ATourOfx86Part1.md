# A Tour of x86 Part 1 Writeup

The challenge reads:  
![tour of x86 part 1](https://user-images.githubusercontent.com/41026969/50127555-a7079d00-023f-11e9-922f-d5880475e4fe.png)

Note: All answers will start with ```0x``` followed by the adress asked. The number of bytes that are required to input were told for  
each question.

You can find the files given in the challenge above the Readme.  

The ```nc``` will prompt the user and ask several questions regarding the ```stage-1.asm``` file. Users can either use ```cat```  to read the file or open with a text editor (I personally prefer sublime text). Upon entering the question correct, it will lead to the next question until the end. When all the answers were answered correctly, the flag will be given.

So throughout the ```stage-1.asm``` file there are examples of assembly instructions. Most of which have to do with memory addresses. Reading it, there are several parts with an arrow and question number, ex: ```<- Question 1```. This will denote what
the question refers to when asked from the ```nc``` command (not open anymore).

### Question 1
Upon doing the ```nc``` command, the first question will ask: ```what is the value of dh after line 129 executes (One byte)``` 

When I first did the challenge, I was held back (time wise) on the formatting of the answers: specifically how many bytes are
represented per hex digit. According to the [hexademical wiki](https://en.wikipedia.org/wiki/Hexadecimal) ```Each hexadecimal digit represents four binary digits, also known as a nibble, which is half a byte.``` So since each hex digit is half a byte, we'll have to input two digits after ```0x``` (according to the question, the answer is one byte). ```0x``` is just a way of saying the next
digits are hexadecimal values and does not count toward the byte count.

Reading line 129 in the ```stage-1.asm```, the assembly line is ```xor dh, dh```. This means we'll do an exclusive-or (also denoted as XOR) comparison with the higher half of register ```d``` (this is what the ```h``` means in ```dh``` - that we're referring to the higher half of d's address).

When we compare using [XOR](https://en.wikipedia.org/wiki/Exclusive_or) we check each bit and if it's the same (1 and 1, or 0 and 0) then it will result in 0. Though if it's different (1 and 0, or 0 and 1), then it will result in 1. (this is where 'exclusive' comes from, basically the bits have to be different).

So since we are XOR'ing the same address, it doens't matter what the actual address is, all the digits will be the same. And as explained above, when we compare two of the same addresses, it will result in 0. Thus, after line 129 executes, the value of dh will simply be all 0's.  

And since the question specifies that we will have the answer in one byte (and we know each hex digit is half a byte), the resulting answer is: ```0x00```

### Question 2  
Upon answering the first question correctly, the second question will be prompted: ```What is the value of gs after line 145 executes? (one byte)```

Line 145 reads: ```mov gs, dx```

So there are a couple hints regarding how to solve this question.

So the first way is that looking a bit above: specifically on line 134, it says ```cmp dx, 0```. ```cmp``` stands for compare
and it will compare the two numerical data fields. Depending on the result, the next line will execute or not. In this case, the 
next line reads: ```jne .death```. First, ```jne``` means "jump not equal." Meaning if the comparison is not equal, it will jump to the address/function named ```.death```. Assuming from the name, this will execute the program. Thus, putting together:

```
cmp dx, 0
jne .death
```

Translated into English: We compare register ```dx``` with 0, and we will jump, if they're not equal, to the function ```.death```

So if the program continues and doesn't die, we'll get to the line in question 2 (145) which reads: ```mov gs, dx```. 
```mov``` stands for "move." It basically means we'll copy the address stored (in this case) ```dx``` to ```gs```. Since we  
know dx is 0 (if it wasn't 0, the program would die/exit from the assembly code above), we're moving the address stored in
```dx``` (all 0's) into ```gs```. So ```gs``` will have all 0s.

Since it is a one byte answer format, we'll write out two hex digits (each hex digit equals one half of a byte as explained
above). Thus the answer for question 2 is: ```0x00```.

A second hint is the comment on the side of the x86 assembly code block (lines 142 - 146). It reads ```Oh yea so this since
the other registers are already 0, I'm just going to use them to help me clear these registers out.``` From that comment
you can assume that by ```clear these registeres out``` the author means he'll set everything to 0. Thus yielding the answer
of: ```0x00```.

### Question 3
When answering the second question correctly: ```What is the value of si after line 151 executes? (two bytes)```

So this question requires a tad bit of reversing. So first line 151 says: ```mov si, sp ;```. And the question is asking
what is the value of ```si```. And to know what value was moved into ```si``` we need to find the value of ```sp```. 
Traversing up the code to investigate the value of ```sp``` we get to line 149 that says ```mov sp, cx```. So now we know
that the value of ```cx``` was moved into ```sp```, and the value moved into ```sp``` was moved into ```si```. So now we need to 
figure out what ```cx``` is equal to. And we know that whole code block from line 142-146 is just setting all the reigsters to 0
because of the comment on the side: ```Oh yea so this since the other registers are already 0, I'm just going to use them to help me
clear these registers out.``` And in those registers is ```cx``` so it's safe to assume that ```cx``` is simply equal to 0. 

If that doesn't convince you, we can also see that on line 107 that the program moves 0 into ```cx```. It reads: ```mov cx, 0```.

Since the answer format needs to be in two bytes, the answer is: ```0x0000```

### Question 4
This question/section revolves around converting to hex, representing as bytes, and learning the higher/lower bits of a register. In
this case we'll look at the lower, and higher bits of register ```a```. It goes into some detail starting at line 77. We assume
that the registers in this file are presenting 16 bit registers. To explain it myself: register ```a``` can look like this:
```
00000000  00000000
There are a total of sixteen 0's, split into 2 halves, both sides have eight 0's
```
(I say "can look like this" becase not all of the time will they all equal 0). Anyway, so each '0' represents a bit. And there are
8 bits in a byte. So to put in graphically:
```
00000000  00000000
^------^  ^------^
1st Byte  2nd Byte
```  
In addition the ```a``` register can be referred to as ```al```, ```ah```, or ```ax``` meaning a lower, a higher, and an all.
This is how it looks visually:
```
00000000    00000000
^------^    ^------^
ah(higher)  al(lower)

ALL of them combined (all 16 bits) is referred to as ax
```
Now we can get into the question, question #4 says: ```What is the value of ax after line 169 executes? (two bytes)```  

Line 169 says: ```mov ah, 0x0e ```

So we know the value ```0x0e``` is going to be places into the upper half of the ```a``` register (```ah```). But the question is  
asking for the value of the whole register (the higher half, AND the lower half). So now we have to traverse up the x86 instructions  to figure out the value of the lower half...

Right above, on Line 168, we find what's being moved into ```al```. The line instruction says: ```mov al, 't'```. So now we know the  lower half is the equivalent of ```t```.

Since the challenge has us give them the address in hex, we need to figure out what ```t``` is in hex. Looking it up on google we
see:

![t in hex](https://user-images.githubusercontent.com/41026969/50204810-0f37ab00-0334-11e9-8e13-142439bed7e1.png)

So thus, the address of the lower half (```al```) is ```0x74```.

Combining the higher half (```ah```) and the lower half (```al```) we get the answer:```ax``` = ```0x0e74```.

NOTE: If you look up the hex values online, be sure to get the capitalization correctly as the hex value for ```T``` is different  
than ```t```.

### Question 5 (the last!!)
The last question asks us: ```What is the value of ax after line 199 executes for the first time? (two bytes)```

Line 199 says: ```mov ah, 0x0e ```. This is line is the exact same from the previous question. So now we know that the
higher half is ```0x0e``` and we just need to find what the lower half is, thus we scan the x86 assembly around for hints.

We see that we are inside a function that serves as a loop - we notice that toward the end of the code, there is a line
that says ```jmp .print_char_loop```, searching through the code (I did CNTRL + F on sublime text) we see that we are 
redirected to the beginning of the funcion that we are in! And we also see that before we jump back to the top of the function, a value is incremented. The function ```.print_char_loop``` is part of the ```print_string``` function.

This part of ```stage-1.asm``` is kinda hard to read (the whitespace/spacing came out weird) and it was hard to understand
the comments given (in my opinion). I'll do my best to explain it better.

So how the function ```print_string``` works is that it takes in a string argument. This string argument is then passed into the
```.print_char_loop``` where as the name suggests, it will print each char, one by one. Inside the char function, on line 197 it says ```mov al, [si]``` which means on the LOWER part of register ```a``` it will equal the first char that was passed into the function. At the end of the ```.print_char_loop``` the value of ```[si]``` is incremented so that the next char will print out. The backets mean "the adress of." It's kind of like C, for example, if you want to print a char in C, it'd look something like: ```print(string[1])```. 

So essentially, in the first loop, the value of ```[si]``` is the first letter (of the string that was passed into the function). And since we see the command: ```mov al, [si]``` we know that the first letter is passed into ```al```.

Looking above, we see the string ```acOS...``` (didn't list it all as it doesn't really matter) is first passed into the function.

The first char of the string is ```a``` and thus it's passed into ```al``` in the first loop. Remember: the question specifies what's the value of ```ax``` ```the first time```. So in the first loop, 'a' is passed into ```al``` and the hex value of 'a' is ```0x61```. Thus ```al``` = ```0x61```. On line 199, it always sets the value of ```ah``` to ```0x0e``` (the line specifically says: ``` mov ah, 0x0e ```).  Thus combining the two halves we get the final answer!

```ax``` = ```0x0e61```

# The flag
Upon answering the last question correctly, we get the flag!

flag: ```flag{rev_up_y0ur_3ng1nes_reeeeeeeeeeeeecruit5!}```

