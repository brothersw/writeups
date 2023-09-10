**Description**

Author: Alex Fulton / Palash Oswal

Smash the stack Let's start off simple, can you overflow the correct buffer? The program is available here. You can view source here. And connect with it using: nc saturn.picoctf.net 64712

Hints
1) How can you trigger the flag to print?
2) If you try to do the math by hand, maybe try and add a few more characters. Sometimes there are things you aren't expecting.
3) Run ```man gets``` and read the BUGS section. How many characters can the program really read?

**Tools**

The main concept of a [buffer overflow](https://en.wikipedia.org/wiki/Buffer_overflow) attack is to get the computer to put more bits into a variable than that variable can hold in the stack memory of that program. 
To preform this attack a basic understanding of how memory is stored in the [stack](https://en.wikipedia.org/wiki/Stack_(abstract_data_type)) and how [assembly](https://en.wikipedia.org/wiki/Assembly_language) and [C](https://en.wikipedia.org/wiki/The_C_Programming_Language) can manipulate the stack. It is crucial to understand future challenges what is going on in this exploit to complete future exploits in the competition, however a crude understanding and playing around with the program can get you the flag in this challenge.

In this writeup and subsequent binary exploitation writeups, I will be using the tools [Ghidra](https://github.com/NationalSecurityAgency/ghidra), [GDB](https://en.wikipedia.org/wiki/GNU_Debugger), and an extension for GDB called [pwndbg](https://github.com/pwndbg/pwndbg).
This tools together provide excellent static and dynamic analysis of a binary. 
Another tool to look at is radare2, however I did not use it for this writeup.
This competition is an excellent place to learn how to use these programs, however I will not be teaching them in entirety.

**Research**

Our local binary isn't marked as executable after downloading this. Lets fix this ```chmod +x vuln```
Use the pre-compiled binary instead of compiling it from source, these binaries use special compiler flags to stop the compiler from protecting the developer from their own mistakes while programming.

Lets look at the source code of the file provided:

```C
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <signal.h>

#define FLAGSIZE_MAX 64

char flag[FLAGSIZE_MAX];

void sigsegv_handler(int sig) {
  printf("%s\n", flag);
  fflush(stdout);
  exit(1);
}

void vuln(char *input){
  char buf2[16];
  strcpy(buf2, input);
}

int main(int argc, char **argv){
  
  FILE *f = fopen("flag.txt","r");
  if (f == NULL) {
    printf("%s %s", "Please create 'flag.txt' in this directory with your",
                    "own debugging flag.\n");
    exit(0);
  }
  
  fgets(flag,FLAGSIZE_MAX,f);
  signal(SIGSEGV, sigsegv_handler); // Set up signal handler
  
  gid_t gid = getegid();
  setresgid(gid, gid, gid);


  printf("Input: ");
  fflush(stdout);
  char buf1[100];
  gets(buf1); 
  vuln(buf1);
  printf("The program will exit now\n");
  return 0;
}
```

First, we need to make a flag.txt file in the same directory as vuln with a test flag to preform this locally, create a test flag with a set of characters you can remember as your test flag. I am using ```testFlag{zyzyzyzy}```

```gets``` is a vulnerable function within C that should not be used for development of any project, see ```fgets``` for developing a program. The binary exploitation series of challenges demonstrates why gets is so dangerous.
This program uses gets, so it is something to look into. Gets will read characters into memory past the buffer of that variable causing the easiest way to have a buffer overflow.

Naive or accidental buffer overflows will often return a SIGSEGV fault as you overwrite something crucial and crash the program, this will halt execution unless a SIGSEGV handler is written to deal with this issue.
Lets look a bit closer at the program, and look at that: 

```signal(SIGSEGV, sigsegv_handler);``` 

The function sigsev_handler is set up as how to deal with a SIGSEGV fault! This function also happens to print the flag!

**Exploit**

Look at where gets stores its input. It stores them in the character array ```buf1```, which has a size of 100. This means that we need to store more than 100 characters in buf1 and then overwrite something critical to the program to cause SIGSEGV to happen, and our flag to print. Lets use \*python2 to input characters into the program to make this easier and pipe that input into the vuln program.

```python2 -c 'print("a"*150)' | ./vuln```
Look at that, it prints the test flag! ```testFlag{zyzyzyzy}```

Now, lets try and do this to the actual server given in the challenge description

```python2 -c 'print("a"*150)' | nc saturn.picoctf.net 64712```

We then get our flag! ```picoCTF{ov3rfl0ws_ar3nt_that_bad_81929e72}```


\*python2 deals with characters as binary in a way that is more familiar to me. It will input characters into stack the way that I expect them too.
