Side Channel is a challenge where they give you a pin code checker to reverse and extract the pin.
The pin code must then be submitted to the challenge server where you will recieve your flag.
Depicted is my problem analysis and solution

**Description**

Author: Anish Singhani

There's something fishy about this PIN-code checker, can you figure out the PIN and get the flag? Download the PIN checker program here pin_checker Once you've figured out the PIN (and gotten the checker program to accept it), connect to the master server using nc saturn.picoctf.net 50562 and provide it the PIN to get your flag.

Hints
1) Read about "timing-based side-channel attacks."
2) Attempting to reverse-engineer or exploit the binary won't help you, you can figure out the PIN just by interacting with it and measuring certain properties about it.
3) Don't run your attacks against the master server, it is secured against them. The PIN code you get from the pin_checker binary is the same as the one for the master server.


**Initial Evaluation**

A side chennel attack is where the user doesn't analyze the cryptographic method itself to break the cryptography, but some other metric such as compute time or operatons required. A weakness in the implementation rather than the cryptographic system itself is required. 
For this challenge, according to hint 1, a timing-based side channel attack needs to be preformed, so we will need to measure compute time.
Running the pin_checker file for the first time shows that this pin must be 8 digits long.


**Analysis**

To make an attack on the timing, we must make a program to analyze timing of the program. I will be using python for this.
To measure compute time, we can use the time module in python. First, we must make a function to analyst the time a given function takes to run.

```python
import time, os
def programTime(x):
    print(x)
    start = time.time()
    run(x)
    print(time.time()-start)
```

Now, we must make a function that runs the pin_checker program

```python
def pin(x):
    os.system("echo '" + str(x) + "' | ./pin_checker")
```

Finally construct inputs to test on the pin_checker program

```python
x = 8

programTime("1"*x)
programTime("2"*x)
programTime("3"*x)
programTime("4"*x)
programTime("5"*x)
programTime("6"*x)
programTime("7"*x)
programTime("8"*x)
programTime("9"*x)
programTime("0"*x)
```

After running this, we can see that the input with 4's takes significantly longer than the rest. For the sake of simplicity, we will try if the function iterates through all of the numbers one by one first. If this doesn't work, we can try other orderings of numbers. If we try any strings with 4's as the first character they take longer than the rest, which confirms that the program takes in inputs one at a time and evaluates by each character one by one. We can determine from this that 4 is the first character in the pin 4\*\*\*\*\*\*\*. Modify the code a bit to test inputs with a 4 at the start, and then 7 more digits.


```python
y = "4"
x = 7
programTime(y + "1"*x)
programTime(y + "2"*x)
programTime(y + "3"*x)
programTime(y + "4"*x)
programTime(y + "5"*x)
programTime(y + "6"*x)
programTime(y + "7"*x)
programTime(y + "8"*x)
programTime(y + "9"*x)
programTime(y + "0"*x)
```

Now the input of 48\*\*\*\*\*\* takes longer. Therefore, that is the next two digits of the pin.
Repeat this process for all inputs. In this case, the pin was 48390513 after checking. Finally, we can check it on the actual server instead of on our local copy of the file, and it returns the flag!
