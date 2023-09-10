**Description**

Author: LT 'syreal' Jones

Can you get the flag? Reverse engineer this binary.

Hints
(none)

**Tools**

I will be using the tools [Ghidra](https://github.com/NationalSecurityAgency/ghidra), [GDB](https://en.wikipedia.org/wiki/GNU_Debugger), and an extension for GDB called [pwndbg](https://github.com/pwndbg/pwndbg).
This tools together provide excellent static and dynamic analysis of a binary. 

**Solve**

Open the file in Ghidra to find the entry, and see a decompiled overview

![entry](https://user-images.githubusercontent.com/59877252/161559147-b550bb36-90df-4941-9ef6-112d05ffdb14.png)

The user input is sent to a checker function at ```0x00101209```. Lets go there and see if we can get a key or an address to watch in GDB that could store the key at some point in time.

![check](https://user-images.githubusercontent.com/59877252/161560585-50bfc6e6-c08f-48d5-b38a-4c88710051ee.png)

It seems that the user input is checked against a string of characters at ```0x00101450``` The each character is stored at RBP-0x30. The input is fully loaded into memory at ```0x00101414```

Lets watch this point while setting a break point at ```0x00101414``` in GDB.

```gdb keygenme```

```start```

![start](https://user-images.githubusercontent.com/59877252/161587595-2443e6fb-0052-4a69-8304-4f0a91579f33.png)

This looks different here!
The addresses look different

GDB handles the addresses differently here. However, the last 3 digits are the same. In Ghidra it enters at ```0x00101120```, and in GDB it enters at ```0x555555555120``` From now on, we can just modify where we want to break at these points. Set your break point at the check that we found, and read the variable from memory.

```break *0x555555555414```

```r```

```x/40x $rbp-0x30```

This returns

```
0x7fffffffde10:	0x6f636970	0x7b465443	0x6e317262	0x30795f67
0x7fffffffde20:	0x305f7275	0x6b5f6e77	0x645f7933	0x63636234
0x7fffffffde30:	0x7d366662	0x00007fff	0x92586800	0x1f04e042
0x7fffffffde40:	0xffffde90	0x00007fff	0x555554e2	0x00005555
0x7fffffffde50:	0xffffdf88	0x00007fff	0x5555556d	0x00000001
0x7fffffffde60:	0xf7000a61	0x00007fff	0x55555520	0x00005555
0x7fffffffde70:	0x00000000	0x00000000	0x55555120	0x00005555
0x7fffffffde80:	0xffffdf80	0x00007fff	0x92586800	0x1f04e042
0x7fffffffde90:	0x00000000	0x00000000	0xf7b030b3	0x00007fff
0x7fffffffdea0:	0x00000001	0x00000000	0xffffdf88	0x00007fff
```

Decode this hex with little endianess, and from hex to unicode

The start of this memory dump returns our flag!
It is: ```picoCTF{br1ng_y0ur_0wn_k3y_d4bccbf6}```


