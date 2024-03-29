# Linear Congruential Generator

LCG is a basic known-seed Pseudo-Random Number Generator which follows this mathematical formula:

$$Xn+1​=(aXn​+c)modm$$

where:

* X is the sequence of pseudo-random values,
* $$m$$ is the modulus,
* $$a$$ is the multiplier,
* $$c$$ is the increment,
* $$X0$$ is the seed or start value.
* $$n$$ represents the step or position in the sequence of pseudorandom numbers being generated.
* $$Xn$$ is the current pseudorandom number at step n.
* $$Xn+1$$ is the next pseudorandom number in the sequence, generated from Xn.



For example, a=5, c=2, m=123, X0=73

The numbers generated by the formula: will be in the range from 0 to m-1.

The seed value initializes the PRNG's internal state. For an LCG, this is the initial value X0  from which the sequence of pseudorandom numbers begins.



Drawback: The PRNG is deterministic, meaning the sequence it generates is entirely determined by the seed. Even though the numbers appear random, if you know the seed and the algorithm's parameters, you can predict the entire sequence of numbers that will be generated. So, it is not very secure.



Here is a code and description above

```
"""
-----
Info
-----

This program implements a Linear Congruential Generator which is based on the formula:
Xn+1 = (aXn + c)mod m

My algorithm is designed as follows:
Xn+1 -> is the next pseudorandom number.
Xn -> is the current pseudorandom number (or the seed for the first iteration)
a is the multiplier (in the program, a = 997 which is a large 3 digit priime number)
c is the increment (in the program, c = 1013904223 which is known to generate good combinations of random numbers)
m is the modulus (in the program, m = 2^{32} which a design decision that balances between maintaining some level of quality in the pseudorandom sequence and exploring the effects of changing other parameters.)
If a range to be generated is given, m changes. Otherwise values between 0 to 2^32-1 is generated.
I wanted to test how keeping a as a 3 digit number would react as opposed to the ANSI C standard a=1664525.

---------------
Program Options
---------------

--run: A flag that must be explicitly set to trigger the generation of numbers. Without this flag, the program shows the help message and exits.
	When --run is provided without any other options, default run is done where seed value is the current time and 10 numbers would be generated
	by default between 1 to 100.
-n or --number: Specifies the number of pseudorandom integers to generate. Defaults to 10 if not provided.
--min: The minimum value in the range of generated integers. Defaults to 1.
--max: The maximum value in the range of generated integers. Defaults to 100.
--seed: An optional seed value for the pseudorandom number generator. If not provided, the current time is used as the seed.

------------------------------
Proof of Algorithm - Math used
------------------------------

The code uses timestamp as a seed by default. Let's say you run the code at 2:30 PM on 24th February 2024, EPOCH timestamp is: 1708785000

Q) Generate numbers between 1 and 100, using timestamp 1708785000 and generate 5 numbers:

root@hex:/home/hex/cryptography# python3 rajpal-lcg.py --run -n 5 --min 1 --max 100 --seed 1708785000
Generated random integers: [8, 91, 18, 5, 16]
root@hex:/home/hex/cryptography# 

Proof:


The math goes like this:
Formula: Xn+1 = (aXn  +c) mod m
a=997
c= 1013904223
X0= 1708785000
X0+1 = X1 = (997*1708785000 + 1013904223) mod 2^32 = 3865500007

Now we have our first random number 3865500007
But we have to scale this between 1 and 100.

Formula for scaling: 1+(X1 * mod100)

Final X1 = 1+(3865500007mod100) = 8
Similarly, Final X2  = 91
Final X3 = 18
Final X4 = 5
Final X5 = 16

So, we can conclude that the Math algorithm in our program is satisfactory and proves.


--------------------
Test Cases and Usage 
--------------------

To test the program, we take these test cases:
1. Default Execution at present time
2. Execution to generate 10 random numbers in between 1 and 100
3. Execution to generate 5 random numbers (No range)
4. Execution to generate only 1 random number between 1 and 7 (LUCKY 7!!)
5. Execution to generate 2 random numbers between 1 and 6 (Dice roll)


Test Case 1:
root@hex:/home/hex/cryptography# python3 rajpal-lcg.py --run
Generated random integers: [3885674302, 970682325, 2416540648, 828277223, 2172574722, 2407384873, 289904140, 2285522971, 3347639430, 1420826941]
root@hex:/home/hex/cryptography# 



Test Case 2:
root@hex:/home/hex/cryptography# python3 rajpal-lcg.py --run -n 10 --min 1 --max 100
Generated random integers: [71, 22, 1, 76, 27, 30, 49, 20, 67, 14]
root@hex:/home/hex/cryptography#


Test Case 3:
root@hex:/home/hex/cryptography# python3 rajpal-lcg.py --run -n 5
Generated random integers: [3885773005, 1069089216, 1743963167, 283426842, 122624161]
root@hex:/home/hex/cryptography#


Test Case 4:
root@hex:/home/hex/cryptography# python3 rajpal-lcg.py --run -n 1 --min 1 --max 7
Generated random integers: [7]
root@hex:/home/hex/cryptography# python3 rajpal-lcg.py --run -n 1 --min 1 --max 7
Generated random integers: [2]
root@hex:/home/hex/cryptography# python3 rajpal-lcg.py --run -n 1 --min 1 --max 7
Generated random integers: [5]
root@hex:/home/hex/cryptography# python3 rajpal-lcg.py --run -n 1 --min 1 --max 7
Generated random integers: [1]
root@hex:/home/hex/cryptography# python3 rajpal-lcg.py --run -n 1 --min 1 --max 7
Generated random integers: [1]
root@hex:/home/hex/cryptography# python3 rajpal-lcg.py --run -n 1 --min 1 --max 7
Generated random integers: [4]
root@hex:/home/hex/cryptography# python3 rajpal-lcg.py --run -n 1 --min 1 --max 7
Generated random integers: [4]
root@hex:/home/hex/cryptography# python3 rajpal-lcg.py --run -n 1 --min 1 --max 7
Generated random integers: [7]
root@hex:/home/hex/cryptography


Test Case 5:
root@hex:/home/hex/cryptography# python3 rajpal-lcg.py --run -n 2 --min 1 --max 6
Generated random integers: [3, 2]
root@hex:/home/hex/cryptography# python3 rajpal-lcg.py --run -n 2 --min 1 --max 6
Generated random integers: [4, 3]
root@hex:/home/hex/cryptography# python3 rajpal-lcg.py --run -n 2 --min 1 --max 6
Generated random integers: [5, 4]
root@hex:/home/hex/cryptography# python3 rajpal-lcg.py --run -n 2 --min 1 --max 6
Generated random integers: [6, 5]
root@hex:/home/hex/cryptography# python3 rajpal-lcg.py --run -n 2 --min 1 --max 6
Generated random integers: [2, 1]
root@hex:/home/hex/cryptography# python3 rajpal-lcg.py --run -n 2 --min 1 --max 6
Generated random integers: [4, 3]
root@hex:/home/hex/cryptography# python3 rajpal-lcg.py --run -n 2 --min 1 --max 6
Generated random integers: [6, 5]
root@hex:/home/hex/cryptography# python3 rajpal-lcg.py --run -n 2 --min 1 --max 6
Generated random integers: [2, 1]
root@hex:/home/hex/cryptography#



----------
Conclusion
----------

Upon testing the program and theorem, I concluded that LCG may be effective using large numbers but still is predictable.
Also, when evaluating lucky 7 or dice rolls, consecutive runs were making prediction possible. For example, 3,2 becomes 4,3 the next second then 5,4 the next
So, LCG is not a very effective algorithm for PRNG


"""

import argparse
import time
import sys

class LCG:
    def __init__(self, seed=None):
        self.a = 997
        self.c = 1013904223
        self.m = 2**32
        if seed is None:
            # Use current time as seed if none provided
            self.state = int(time.time())
        else:
            self.state = seed

    def next(self):
        self.state = (self.a * self.state + self.c) % self.m
        return self.state

    def generate_n_integers(self, n=10, min_val=None, max_val=None):
        """
        Generate n integers, either within a given range [min_val, max_val] or in the full range of 0 to m-1 if min_val and max_val are None.
        """
        random_integers = []
        if min_val is None or max_val is None:
            # Generate integers in the full range of 0 to m-1
            for _ in range(n):
                num = self.next()
                random_integers.append(num)
        else:
            # Generate integers within the specified range [min_val, max_val]
            range_width = max_val - min_val + 1
            for _ in range(n):
                num = min_val + (self.next() % range_width)
                random_integers.append(num)
        return random_integers

def main():
    parser = argparse.ArgumentParser(description='Generate pseudorandom integers using a Linear Congruential Generator.')
    parser.add_argument('--run', action='store_true', help='Run the generator in default state and generate 10 numbers. Without this flag, the program will not generate numbers.')
    parser.add_argument('-n', '--number', type=int, default=10, help='Number of random integers to generate')
    parser.add_argument('--min', type=int, default=None, help='Minimum value of the random integers (optional). Default [0 to 2^32 -1]')
    parser.add_argument('--max', type=int, default=None, help='Maximum value of the random integers (optional). Default [0 to 2^32 -1]')
    parser.add_argument('--seed', type=int, default=None, help='Seed value for the pseudorandom number generator (optional). By default, uses current time')

    args = parser.parse_args()

    if not args.run:
        parser.print_help()
        sys.exit(1)

    lcg = LCG(args.seed)
    random_integers = lcg.generate_n_integers(args.number, args.min, args.max)
    print("Generated random integers:", random_integers)

if __name__ == "__main__":
    main()

```





