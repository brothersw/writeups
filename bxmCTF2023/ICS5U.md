# ICS5U
## REV 2 - Author: Ryzon - Writeup by: brosu
## Description

## First look

There are 3 things here, a clue text file filled with what appears to be some encoded message, a php file named puzzle.php, and a C++ file named main.cpp

Here are the contents of those files:
### main.cpp
```C++
#include <bits/stdc++.h>
using namespace std;

int64_t manipulate(const string &input)
{
	const int32_t SZ = input.size(), B1 = 131, B2 = 13, K = 1e9 + 7;
	int64_t fh[SZ], fs[SZ], pw1[SZ], pw2[SZ];
	for (int i = 0; i < SZ; ++i) fh[i] = fs[i] = 0;
	pw1[0] = pw2[0] = 1;
	for (int _ = 1; _ <= SZ; ++_)
	{
    	pw1[_] = pw1[_-1] * B1 % K;
    	pw2[_] = pw2[_-1] * B2 % K;

    	fh[_] = (fh[_-1] * B1 + input[_-1]) % K;
    	fs[_] = (fs[_-1] * B2 + input[_-1]) % K;
	}

	int64_t f = (fh[SZ] - fh[0] * pw1[SZ] % K + K) % K;
	int64_t s = (fs[SZ] - fs[0] * pw2[SZ] % K + K) % K;

	return (f << 31) ^ s;
}

int main()
{
	while (true)
	{
    	printf("Error. Login Required.\n");
    	printf("Please enter the corresponding passcodes to proceed.\n");

    	int64_t a, b, c, d;

    	printf("Enter \'a\'\n");
    	cin >> a;

    	printf("Enter \'b\'\n");
    	cin >> b;

    	printf("Enter \'c\'\n");
    	cin >> c;

    	printf("Enter \'d\'\n");
    	cin >> d;

    	int64_t x = manipulate(to_string(a));
    	int64_t y = manipulate(to_string(b));
    	int64_t z = manipulate(to_string(c));
    	int64_t w = manipulate(to_string(d));

    	int64_t token = manipulate(to_string(x + y + z + w));
    	cout << "ctf{" << to_string(token) << "}\n";
	}
}
```

### puzzle.php
```php
<?php

/* Hmm, what does this program seem to do? */

function alpha($a, $b)
{
	if ($a < $b)
	{
    	return alpha($b, $a);
	}
    
	if ($a % $b == ($a ^ $a))
	{
    	return $b;
	}
    
	return alpha($b, $a % $b);
}

function beta($x, $y, $z)
{
	$ans = 1;

	if ($y & 1 != 0)
	{
    	$ans = $x;
	}

	while ($y != 0)
	{
    	$y >>= 1;
    	$x = ($x * $x) % $z;

    	if ($y & 1 != 0)
    	{
        	$ans = ($ans * $x) % $z;
    	}
	}

	return $ans;
}

function gamma($n)
{
	for ($_ = 2; $_ <= $n; ++$_)
	{
   	 
    	if (alpha($_, $n) == 1)
    	{
       	 
        	if (beta($_, $n - 1, $n) != 1)
        	{
            	return 0;
        	}
    	}
	}
    
	return 1;
}

function init($n)
{
	$check = (gamma($n) == 1 ? true : false);

	if ($check)
	{
    	echo('YES');
	}

	else
	{
    	echo('NO');
	}
}

?>
```

###Clue.txt
```
JEQHO33OMRSXELROFYQHO2DBOQQGC4TFEB2GQZJAMZUXE43UEBTG65LSEBXHK3LCMVZHGIDPMYQHI2DFEBZWC3LFEB2HS4DFEBQXGIDJNYQCE4DVPJ5GYZJOOBUHAIRAORUGC5BAMFZGKIDHOJSWC5DFOIQHI2DBNYQDCIDUOJUWY3DJN5XC4LRO
```

## Initial analysis

The Clue.txt file is encoded base32 text
It says:
```
I wonder... what are the first four numbers of the same type as in "puzzle.php" that are greater than 1 trillion...
```
This doesn't entirely make sense at the moment, but it seems that we need to solve what type of numbers puzzle.php uses, so we should reverse it.
First, a brief look at main.cpp shows that it takes in 4 numbers. I think we need to solve what this hint means for puzzle.php and feed those four numbers into main.cpp to solve for the flag.

## Reversing the PHP file

Let’s walk through the individual functions and see what they do.

Alpha is a gcd algorithm using the [Euclidean alogirithm](https://en.wikipedia.org/wiki/Euclidean_algorithm)
```php
function gcd($a, $b)
{
	if ($a < $b)
	{
    	return gcd($b, $a);
	}
    
	if ($a % $b == ($a ^ $a))
	{
    	return $b;
	}
    
	return gcd($b, $a % $b);
}
```

Beta is a fast modular exponentiation algorithm using [binary exponentiation](https://cp-algorithms.com/algebra/binary-exp.html)
```php
function beta($x, $y, $z)
{
	$ans = 1;

	if ($y & 1 != 0)
	{
    	$ans = $x;
	}

	while ($y != 0)
	{
    	$y >>= 1;
    	$x = ($x * $x) % $z;

    	if ($y & 1 != 0)
    	{
        	$ans = ($ans * $x) % $z;
    	}
	}

	return $ans;

}
```

No, it is time to figure out what gamma does.
```php
function gamma($n)
{
	for ($_ = 2; $_ <= $n; ++$_)
	{
   	 
    	if (gcd($_, $n) == 1)
    	{
       	 
        	if (binexp($_, $n - 1, $n) != 1)
        	{
            	return 0;
        	}
    	}
	}
    
	return 1;
}
```

Here is a simplified version of the php code written in python
```python3
import math
def checkNum(n):
	for i in range(2, n):
    	if math.gcd(i, n) == 1 and pow(i, n-1, n) != 1:
        	return False
	return True
```

## Op-Sec

First, we should check if this is an exisitng algorithm somewhere.
This checks all numbers 2 through n, and sees if first i and n are relatively prime, and then if i^(n-1) = 1 (mod n) it fails the test.
Initially, I was stuck on the [Miller-Rabin primality test](https://en.wikipedia.org/wiki/Miller%E2%80%93Rabin_primality_test); however this wasn't matching the behavior I was seeing properly.
It turned out to be an algorithm that the Miller-Rabin primality test was derived from, the [Fermat primaility test](https://en.wikipedia.org/wiki/Fermat_primality_test).

## Fermat primality and pseudoprimes

The fermat primality check if a number is probably prime. It does this by having two numbers, a and p. ``a`` is the base, and ``p`` is the number that you are checking for. A must be less than p, a and p must be relatively prime, and a^(p-1) = 1 (mod p).
At random, some bases should be chosen, and then you check your number a few times.
This primality test has quite a few pseudoprimes that appear for each base; however, this decreases the more bases that you check.

Our function models this by checing relative primality of i and n, and then preforming this check between our new base i and p.
It loops through all bases between 2 and p; making this a fermat primality test that checks all bases, eliminating the probabalistic nature of the primality test.

Initially, I tried the the first 4 prime numbers by putting them into main.cpp and getting a flag. This however, was not the correct flag, and so I began to move onto pseudoprimes.

It seems almost impossible to find these numbers. There are infiniate numbers and infinate pseudoprimes; therefore, it stands to reason that some pseudoprimes must exist in all bases.
There are a special kind of [fermat pseudoprimes](https://en.wikipedia.org/wiki/Fermat_pseudoprime) called [Carmichael numbers]({)https://en.wikipedia.org/wiki/Carmichael_number). 

### Brute forcing
My very first attempt at finding these numbers was just to brute force non-prime numbers past one trillion to find these numbers. It was going relatively quickly as you can rule out most of these number in just a few bases; however, after one million numbers none existed in this range.
### Calculation
Some very intelligent mathematicians have charactarized these numbers into a way that makes them relatively easy to find. It will still be a challenge to find out. There is a very good github repository that helps you generate these numbers on your own [here](https://github.com/angzosan/Pseudroprime-and-Carmichael). While this scrips ran, I began a search for a comprehensive list of Carmichael numbers after one trillion.
### Solution
As the script was running, I found a list at [http://s369624816.websitehome.co.uk/rgep/cartable.html](http://s369624816.websitehome.co.uk/rgep/cartable.html). While this is an http site, and there was some apprehension going onto the site, it contained the numbers I was looking for!
Entering the numbers: 1000151515441, 1000321709401, 1000642078801, 1001102784001 got me the flag!

``ctf{2029395652961987395}``
