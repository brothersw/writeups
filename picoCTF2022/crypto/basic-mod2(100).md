**Description**

Author: Will Hong

A new modular challenge! Download the message here. Take each number mod 41 and find the modular inverse for the result. Then map to the following character set: 1-26 are the alphabet, 27-36 are the decimal digits, and 37 is an underscore. Wrap your decrypted message in the picoCTF flag format (i.e. picoCTF{decrypted_message})

Hints
1) Do you know what the modular inverse is?
2) The inverse modulo z of x is the number, y that when multiplied by x is 1 modulo z
3) It's recommended to use a tool to find the modular inverses

**Solving**

To solve this program, I created a program to automate a tedious process, after doing this by hand for [basic-mod1](./crypto/basic-mod1(100).md)

\* **DISCLAIMER** some code I took from [source](https://stackoverflow.com/questions/4798654/modular-multiplicative-inverse-function-in-python) instead of using a tool to find the modular inverses.

First, take the modular mulplicative inverse of the numbers. [Wikipedia](https://en.wikipedia.org/wiki/Modular_multiplicative_inverse)
Then, substitute out the values recieved for the correct number or letter
Then you have the flag!
This program returns 1NV3R53LY_H4RD_261CB530, which becomes picoCTF{1NV3R53LY_H4RD_261CB530}

```python
numbers = "145 167 233 272 344 91 395 393 433 92 77 414 344 318 420 263 87 186 96 103 91 354 161"
numberlist = numbers.split()
print(numberlist)

out = []

def s(x):
    switcher = {
            1:"A",
            2:"B",
            3:"C",
            4:"D",
            5:"E",
            6:"F",
            7:"G",
            8:"H",
            9:"I",
            10:"J",
            11:"K",
            12:"L",
            13:"M",
            14:"N",
            15:"O",
            16:"P",
            17:"Q",
            18:"R",
            19:"S",
            20:"T",
            21:"U",
            22:"V",
            23:"W",
            24:"X",
            25:"Y",
            26:"Z",
            27:"0",
            28:"1",
            29:"2",
            30:"3",
            31:"4",
            32:"5",
            33:"6",
            34:"7",
            35:"8",
            36:"9",
            37:"_"
            }
    return switcher.get(x)

def egcd(a, b):
    if a == 0:
        return (b, 0, 1)
    else:
        g, y, x = egcd(b % a, a)
        return (g, x - (b // a) * y, y)

def modinv(a, m):
    g, x, y = egcd(a, m)
    if g != 1:
        raise Exception('modular inverse does not exist')
    else:
        return x % m

for x in numberlist:
    
    x = int(x)# % 41 
    x = modinv(x, 41)
    print(x)
    out.append(s(x))


print(*out)

outstring = ''

for i in out:
    outstring = outstring + i

print(outstring)
```
