
# Description

Sometime, less ~~instruction~~ equal more ~~instruction~~ ...
Author: bronson113
Writeup by: brosu

``.tar.gz`` and netcat address provided, both seem to contain the same crackme code

# Initial Analysis

Inside the ``.tar.gz`` is a binary called ``lessequalmore`` and a text file containing integers called ``chal.txt``

``lessequalmore: ELF 64-bit LSB pie executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, BuildID[sha1]=0396c66f756be3bbb228b6ccb6005df9bd314504, for GNU/Linux 3.2.0, not stripped``

Pulling the binary up in Ghidra reveals that this is a vm reverse engineering challenge, where some instruction set is defined in a separate file and then that separate file determines the code that runs. 

> In this case reverse engineering is easier since the binary isn't stripped.

The chal.txt file is read into memory, where it loops through and executes the custom instruction set.


# Python Decomp

When chal.txt is read, there is an instruction counter and each instruction has 3 memory regions which I will call A, B, and C. Regions A and B are pointers, while C signifies the next instruction in some circumstances.

Pseudo-code for this process involves:
```
if A >= 0
	use *A
else
	use -input

if B >= 0
	*B -= (*A or input)
else
	print(*A or input)

if 0 >= *B 
	jump to C
else
	continue to next instruction
```

Reversing this process into python results in
```python
mem : list[int] = []

def getChar() -> str:
	a = input("Enter a char: ")
	if(len(a) == 0):
		a = '\n'
	return a[0]

def getBignum() -> int:
	ret : int
	char : str
	char = getChar()
	
	if char == '%':
		char = getChar()
		
		if char == 'x':
			ret = 0
			char = getChar()
			
			while char != '\n':
				ret *= 16
				if ord(char) < ord('A') or ord('F') < ord(char):
					if ord(char) < ord('a') or ord('f') < ord(char):
						ret += ord(char) - 0x30
					else:
						ret += ord(char) - 0x57
				else:
					ret += ord(char) - 0x37
				char = getChar()

		else:
			if ord(char) < ord('y'):
				if char == '%':
					ret = ord(char)
					return -ret
				
				if ord("$") < ord(char) and (ord(char) - 48) < 10:
					ret = ord(char) - 0x30
					char = getChar()
					
					while char != '\n':
						ret = (ord(char) - 0x30) + ret * 10
						char = getChar()
					return -ret

			ret = ord(char)
	else :
		ret = ord(char)

	return -ret


def bignumSub(a : int, b : int) -> int:
	return b - a

def op1(a : int, b : int):
	local_30 : int
	
	isNegative : bool = a < 0
	if not isNegative:
		local_30 = mem[a]
	else:
		local_30 = getBignum()

	isNegative : bool = b < 0
	if not isNegative:
		mem[b] = bignumSub(local_30, mem[b])
	else:
		print(chr(local_30), end='')

def runProg():
	ins : int = 0;
	
	while -1 < ins:
		print(ins)
		op1(mem[ins], mem[ins+1])
		
		if mem[mem[ins + 1]] > 0:
			ins += 3
		else:
			ins = mem[ins+2]


def readProg(filename : str):
	with open(filename, "r") as file:
		for line in file.readlines():
			mem.extend([int(a) for a in line.strip("\n ").split(" ")])

def main():
	readProg("chal.txt")
	runProg()

if __name__ == "__main__":
	main()

```

The bulk of ``getBigNum()`` handles the % modifier on input

# Side Channel Analysis

With over 100000 instruction cycles, reversing this by hand seems infeasible. Trying to get some information from side channel analysis may be useful. Graphing instruction count may be useful.

Changing getChar() to buffer input may be useful

```python
strBuf : str

def getChar() -> str:
	global strBuf
	if len(strBuf) > 0:
		a = strBuf[0]
		strBuf = strBuf[1:]
	else:
		a = '\n'
	return a[0]
```

This allows a change of ``main()``, ``runProg()``, and the addition of a ``plotInstructions()`` function

```python
import matplotlib.pyplot as plt

def runProg() -> int:
	ins : int = 0;
	count : int = 0;

	while -1 < ins:
		count += 1
		op1(mem[ins], mem[ins+1])
		
		if mem[mem[ins + 1]] > 0:
			ins += 3
		else:
			ins = mem[ins+2]
	return count

def plotInstructions(insList : list[int]):
	plt.scatter([i for i in range(len(insList))], insList)
	plt.show()

def main():
	global strBuf
	insList : list[int] = []
	for i in range(0, 100):
		strBuf = "a" * i
		readProg("chal.txt")
		insList.append(runProg())

	print(insList)
	plotInstructions(insList)
```

This leads to the following graph with character count on the x-axis and instruction number on the y-axis:

![character count side channel](https://github.com/brothersw/writeups/assets/59877252/1e4c1cc8-64f6-4c64-a959-9e7a50bc79a5)


This graph shows that characters are grouped and processed into chunks of 8, and there are either 7 or 8 of these chunks.

Looking for a completed block starting with the 7 characters ``hitcon{`` leads to this graph:

![hitcon{r side channel](https://github.com/brothersw/writeups/assets/59877252/274566be-ee57-45e7-92f0-fe9d1ff8f272)


This means that the first chunk is ``hitcon{r``
Doing further side channel analysis leads to the conclusion that instruction count jumps by a significant margin only on a solved chunk, not on individual solved characters. This rules out a brute force approach as brute forcing 8 characters at once in a vm isn't feasable.

We can make these 4 conclusions from this data:
- Characters are grouped into chunks of 8
- There are probably 7 of these chunks to give a flag length of 56, though there is a chance that 8 chunks exist.
- Memory has 3 segments, with each instruction having those 3 distinctive values. Regions A and B are pointers while region C denotes the next instruction in some cases.
- When a chunk is solved, instruction count raises by a detectable margin

# Memory Analysis

The input characters are stored starting at ``mem[16]``. Finding this was relatively easy with trace diffing at different parts of execution.
Transforms are applied to the characters here, and the numbers stored here change from ascii to a dramatically different set of numbers

Removing the side channel stuff from the python code and changing ``main()`` enables analysis of a buffered flag as input characters, and what that transforms into. 
```python
def main():
	global strBuf
	global flag
	strBuf = 'hitcon{r' + 'aaaaaaaa' + 'aaaaaaab' + 'aaaaaaaa' + 'aaaaaaaa' + 'aaaaaaaa' + 'aaaaaaaa' + 'aaaaaaaa'
	flag = strBuf
	readProg("chal.txt")
	runProg()
	for i in range(16, len(flag) + 16):
		if i % 8 == 0:
			print(f"\nBlock: {int((i-16) / 8)} = {flag[i-16:i-8]}")
		print(mem[i], end=' ')
```

Playing with this code shows that all blocks are evaluated the same way, and all change independently one one another. Also, ``hitcon{r`` transforms into ``16774200 1411 16776275 3646 1532 6451 2510 16777141``. This also exists in the chal.txt file elsewhere! 

This is with a set of 8 different similar sets of numbers, confirming that we have 8 blocks of 8 characters!

```
[16774200, 1411, 16776275, 3646, 1532, 6451, 2510, 16777141]
[16775256, 2061, 16776706, 2260, 2107, 6124, 878, 16776140]
[16775299, 1374, 16776956, 2212, 1577, 4993, 1351, 16777040]
[16774665, 1498, 16776379, 3062, 1593, 5966, 1924, 16776815]
[16774318, 851, 16775763, 3663, 711, 5193, 2591, 16777069]
[16774005, 1189, 16776283, 3892, 1372, 6362, 2910, 307]
[16775169, 1031, 16776798, 2426, 1171, 4570, 1728, 33]
[16775201, 819, 16776898, 2370, 1132, 4255, 1900, 347]
```

# Transform Emulation

Using this memory analysis some more shows that changing the character in each column of its chunk by one ascii value leads to a shift in the resulting transformed number list.

Changing the last character from a to b for example leads to a change of
``-7	-3	+6	+7	+4	+8	+13	+15`` to the transformed output. The transformed output is a unsigned 24 bit integer and will roll over if it goes too high or becomes negative.

Here are all of the shifts and their associated with an increase of one ascii value.
```
unsigned integer rollover at 2^24

0	0	0	0	0	0	0	+1
-7	-3	6	7	4	8	13	15	

0	0	0	0	0	0	+1	0
-2	11	-2	2	9	20	-6	-11

0	0	0	0	0	+1	0	0
-13	-10	-5	17	-8	1	22	15

0	0	0	0	+1	0	0	0
4 	6 	4 	-6 	6 	5 	-8 	-5

0	0	0	+1	0	0	0	0
-4 	5 	-3 	5 	3 	13 	1 	-6

0	0	+1	0	0	0	0	0
3 	2 	-3 	-3 	-1 	-2 	-6 	-8

0	+1	0	0	0	0	0	0
-2 	3 	-4 	3 	2 	7 	-1 	-5 

+1	0	0	0	0	0	0	0
-7 	-2 	-2 	9 	-2 	6 	9 	5 
```

Simulating this in python results in

```python
import numpy as np

def predictEncoding(chunk : str):
	assert len(chunk) == 8
	out = np.zeros(8, dtype=int)
	c1 = np.array([-7, -2, -2, 9, -2, 6, 9, 5]) * ord(chunk[0])
	c2 = np.array([-2, 3, -4, 3, 2, 7, -1, -5]) * ord(chunk[1])
	c3 = np.array([3, 2, -3, -3, -1, -2, -6, -8]) * ord(chunk[2])
	c4 = np.array([-4, 5, -3, 5, 3, 13, 1, -6]) * ord(chunk[3])
	c5 = np.array([4, 6, 4, -6, 6, 5, -8, -5]) * ord(chunk[4])
	c6 = np.array([-13, -10, -5, 17, -8, 1, 22, 15]) * ord(chunk[5])
	c7 = np.array([-2, 11, -2, 2, 9, 20, -6, -11]) * ord(chunk[6])
	c8 = np.array([-7, -3, 6, 7, 4, 8, 13, 15]) * ord(chunk[7])

	out += c1
	out += c2
	out += c3
	out += c4
	out += c5
	out += c6
	out += c7
	out += c8

	for i in range(len(out)):
		out[i] %= (2**24)
	return out
```

Testing for ``predictEncoding('hitcon{r')`` gives the desired output of ``[16774200, 1411, 16776275, 3646, 1532, 6451, 2510, 16777141]`` which is a pleasant surprise that no static offsets are needed to get the desired output.

# Z3 Solve Script

Now this problem just needs to be done in reverse. z3 is a powerful theorem proving tool that can do this task. Rotating the array from earlier relates each character to a chunk's values. Using symbolic BitVecotrs to show this relationship, z3 can give us the characters that are desired.

```python
from z3 import *

def z3Decode(encodedChunk : list[int]):
	# Define the symbolic variables for the chunk
	chunk = [BitVec(f'chunk_{i}', 8) for i in range(8)]

	# Create a Z3 solver
	solver = Solver()

	o1 = (-7 * chunk[0] + -2 * chunk[1] + 3 * chunk[2] + -4 * chunk[3] + 4 * chunk[4] + -13 * chunk[5] + -2 * chunk[6] + -7 * chunk[7]) % (2**24) == encodedChunk[0]
	o2 = (-2 * chunk[0] + 3 * chunk[1] + 2 * chunk[2] + 5 * chunk[3] + 6 * chunk[4] + -10 * chunk[5] + 11 * chunk[6] + -3 * chunk[7]) % (2**24) == encodedChunk[1]
	o3 = (-2 * chunk[0] + -4 * chunk[1] + -3 * chunk[2] + -3 * chunk[3] + 4 * chunk[4] + -5 * chunk[5] + -2 * chunk[6] + 6 * chunk[7]) % (2**24) == encodedChunk[2]
	o4 = (9 * chunk[0] + 3 * chunk[1] + -3 * chunk[2] + 5 * chunk[3] + -6 * chunk[4] + 17 * chunk[5] + 2 * chunk[6] + 7 * chunk[7]) % (2**24) == encodedChunk[3]
	o5 = (-2 * chunk[0] + 2 * chunk[1] + -1 * chunk[2] + 3 * chunk[3] + 6 * chunk[4] + -8 * chunk[5] + 9 * chunk[6] + 4 * chunk[7]) % (2**24) == encodedChunk[4]
	o6 = (6 * chunk[0] + 7 * chunk[1] + -2 * chunk[2] + 13 * chunk[3] + 5 * chunk[4] + 1 * chunk[5] + 20 * chunk[6] + 8 * chunk[7]) % (2**24) == encodedChunk[5]
	o7 = (9 * chunk[0] + -1 * chunk[1] + -6 * chunk[2] + 1 * chunk[3] + -8 * chunk[4] + 22 * chunk[5] + -6 * chunk[6] + 13 * chunk[7]) % (2**24) == encodedChunk[6]
	o8 = (5 * chunk[0] + -5 * chunk[1] + -8 * chunk[2] + -6 * chunk[3] + -5 * chunk[4] + 15 * chunk[5] + -11 * chunk[6] + 15 * chunk[7]) % (2**24) == encodedChunk[7]

	solver.add(o1, o2, o3, o4, o5, o6, o7, o8)
	
	if solver.check() == sat:
	    model = solver.model()
	    original_chunk = [chr(model[c].as_long()) for c in chunk]
	    original_chunk = ''.join(original_chunk)
	    print("Original chunk:", original_chunk)
	else:
	    print("No solution found.")
```

Testing with ``z3Decode(predictEncoding('hitcon{r'))`` gives out ``hitcon{r`` which is exactly the right input. Moving on to test all of the comparison values gives the flag.

```python
z3Decode([16774200, 1411, 16776275, 3646, 1532, 6451, 2510, 16777141])
z3Decode([16775256, 2061, 16776706, 2260, 2107, 6124, 878, 16776140])
z3Decode([16775299, 1374, 16776956, 2212, 1577, 4993, 1351, 16777040])
z3Decode([16774665, 1498, 16776379, 3062, 1593, 5966, 1924, 16776815])
z3Decode([16774318, 851, 16775763, 3663, 711, 5193, 2591, 16777069])
z3Decode([16774005, 1189, 16776283, 3892, 1372, 6362, 2910, 307])
z3Decode([16775169, 1031, 16776798, 2426, 1171, 4570, 1728, 33])
z3Decode([16775201, 819, 16776898, 2370, 1132, 4255, 1900, 347])
```

The completed flag is ``hitcon{r3vErs1ng_0n3_1ns7ruction_vm_1s_Ann0ying_c9adf98b67af517}``!


