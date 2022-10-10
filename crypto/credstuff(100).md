**Description**

Author: Will Hong / LT 'syreal' Jones

We found a leak of a blackmarket website's login credentials. Can you find the password of the user cultiris and successfully decrypt it? Download the leak here. The first user in usernames.txt corresponds to the first password in passwords.txt. The second user corresponds to the second password, and so on.

Hints
1) Maybe other passwords will have hints about the leak?

**Solving**

The user "cultiris" is the 378th line on the file, which corresponds with "cvpbPGS{P7e1S_54I35_71Z3}" in passwords.txt.
We already know we are on the correct track, however, cvpbPGS{ != picoCTF{. This already looks like some kind of substitution cipher, but lets look at passwords.txt more closely to see if there are any hints. Running the command "grep pico" on the file returns "pICo7rYpiCoU51N6PicOr0t13", and something is interesting here. The last 5 characters "r0t13" is the name of a type of cipher. 
Rot13, or the Ceasar cipher is a common cipher for text. This is where you shift each letter in the alphabet by 13 characters to get another string. Lets try and see if this gets us the flag.
It does! Our flag is picoCTF{C7r1F_54V35_71M3} after decoding.
