# Gametime writeup - 50 points

This is what the challenge description says:

![gametime](https://user-images.githubusercontent.com/41026969/51548489-6569a380-1e36-11e9-85a1-b46d35d94fe3.png)

First I played around with the executable, though since I'm on a linux system I have to use [wine](https://www.winehq.org/) to run it since it's a windows executable ```gametime.exe: PE32 executable (console) Intel 80386, for MS Windows```.

![gametime](https://user-images.githubusercontent.com/41026969/51547995-59c9ad00-1e35-11e9-8f06-083972a1d4a8.png)

Basically how this game works is that you have to press a specific character when one is printed out, if you fail to press the character at a certain timing, the game will exit and you won't get the flag. 

### Reverse engineering the executable 

I open up IDA 32 bit (since this executable is 32 bit) and start reversing.

First thing I like to do when doing these reverse engineering chellanges is to open up the strings page. This is similar to using the ```strings``` command on Linux except you can see addresses and cross reference them. To open a strings page, on the top click view > open subviews > strings.

![gametime](https://user-images.githubusercontent.com/41026969/51716876-9e5b7100-200c-11e9-87ec-fc1ca89ef7c9.PNG)

Looking at the strings page, I see the failure message occur two separate times. We can click the string and then again click the ```DATA XREF SUB...``` to go to the function in proximity view. Below is a picture of one of the functions with the error messages:

![gametime](https://user-images.githubusercontent.com/41026969/51717048-4a9d5780-200d-11e9-9a0b-52886c9f7c3b.PNG)

Both these functions have a ```jnz``` command before the ```UDDER FAILURE!``` message. We can change this jump not zero command to jump zero so that we never get the failure message.

### Patching the program
I'm going to show how to patch one of the functions (there's a total of two you need to patch) in IDA pro. We want to patch the ```jnz``` on the function right above the failure function to ```jz```.

1) Highlight the jz that we want to change
2) On the top, Edit > Patch program > Change byte
3) Looking up how to change [jnz to jz or vice versa](https://stackoverflow.com/questions/12039220/how-does-one-change-an-instruction-with-a-hex-editor) we see that we have to change the first number ```75``` to ```74```, simply just delete it and type in 74. it should look like this:

![gametime](https://user-images.githubusercontent.com/41026969/51728018-99161a80-203c-11e9-8260-643a7903dc68.PNG)

4) Click Ok
5) You can then run it using IDA's debugger or apply the patches to the executable shown below:
6) Edit > Patch program > Apply patches to input file. If you want to create a backup click the checkbox next to it, then click Ok. Now when you run the program (even without IDA) it will be the patched version.

### The flag
So after applying the patches, we don't need to do anything when we run the game, just wait for it and wait for the key.

Doing so spits out flag: ```key is  (no5c30416d6cf52638460377995c6a8cf5)```
