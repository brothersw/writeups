**Description**

Author: Will Hong

We found this weird message being passed around on the servers, we think we have a working decrpytion scheme. Download the message here. Take each number mod 37 and map it to the following character set: 0-25 is the alphabet (uppercase), 26-35 are the decimal digits, and 36 is an underscore. Wrap your decrypted message in the picoCTF flag format (i.e. picoCTF{decrypted_message})

Hints
1) Do you know what mod 37 means?
2) mod 37 means modulo 37. It gives the remainder of a number after being divided by 37.

**Decoding**

Use [modular arithmetic](https://en.wikipedia.org/wiki/Modular_arithmetic) to transform the numbers given in message.txt in a mod 37 environment. 

i.e **38≡1 (mod 37)** or **40≡3 (mod 37)**

The number is then substituted with the character set above where 0-25 is the alphabet A-Z, and 26-24 is 0-9 and 35 is _

You can do this by hand or with a custom program to make it a bit faster if one is fluent in a programming lanugage.
This decoding is the flag!
