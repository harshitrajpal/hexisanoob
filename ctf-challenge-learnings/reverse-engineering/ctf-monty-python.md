# CTF: Monty Python

The CTF challenge required answers to 3 questions like in Monty Python's bridge of death.

<figure><img src="../../.gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>

Upon decompiling in Ghidra, we see the questions and conditions upon which flag would be returned:

<figure><img src="../../.gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>



As we can see, to get the flag (print\_flag()) we need to make sure that all three conditions on the questions are wrong otherwise the program would exit using the "throw\_into\_gorge\_of\_eternal\_peril()" function.



Let's see question1:

<figure><img src="../../.gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>



Upon entering the string mentioned here, strcmp works and question 1 would be answered correctly.

Let's see question2:

<figure><img src="../../.gitbook/assets/image (3).png" alt=""><figcaption></figcaption></figure>

Hmmm, 2 variables are input and func2 is called and variable 1 is assigned it's value. Function returns true when a is not equal to b. It returns false when a is equal to b. As in the main() we can see that the question2() must return 0.

<figure><img src="../../.gitbook/assets/image (4).png" alt=""><figcaption></figcaption></figure>

So, to make question2 return 0, variable 1 and variable 2 in the function should be equal. Which means after func2() is done, variable 1 should remain the same.

Let's see func2 now.

<figure><img src="../../.gitbook/assets/image (5).png" alt=""><figcaption></figcaption></figure>

Inspect the three conditions here. When func2 is called from question2(), 3 parameters are sent: variable1, 0 and 0x14 which is 20 in decimal. So, the value of variable 1 and variable 2 must lie in \[0,20].

Finally in func2 we can see the main operating condition.

iVar1 = 0 + (20-0)/2

if param\_1 (which is variable 1 in question2()) is less than 10, value changes. If it is greater than 10, value changes. Only way value remains same is if variable 1 in question2() is 10. And by extension variable 2 should be 10 too to make the 2nd condition in main() false and allow further execution.

Thus: 10 and 10

Finally, question3() should return 0 to continue execution and print flag.

Here is what question 3 looked like:

<figure><img src="../../.gitbook/assets/image (6).png" alt=""><figcaption></figcaption></figure>



For simplicity, I rewrote this using simple variable names:

```c
int question3(void)

{
  long a;
  int b;
  int c;
  int d;
  int i;
  
  a = *(long *)(in_FS_OFFSET + 0x28);
  i = 1;
  do {
    if (9 < i) {
      d = 0;
      }
      return d;
    }
    b = get_number();
    c = get_number();
    if (((0xff < b) || (0xff < c)) ||
       (i != (char)forestOfEwing[(long)b * 0x100 + (long)c])) {
      d = 1;
    }
    i = i + 1;
  } while( true );
}
```

Here, we can see for this function to return 0, we need to make d=0. For that to happen i needs to be 10. So, for 9 executions we have to make this: `"if (((0xff < b) || (0xff < c)) || (i != (char)forestOfEwing[(long)b * 0x100 + (long)c]))`" condition false 9 times.

Now, it is easy to provide b and c with inputs less than 0xff (255 in decimal) but the third or condition is tricky. We need to find elements in the array forestOfEwing at indices b\*256 + c == index which is equal to i's current value.

Upon inspecting the array, we see this:

<figure><img src="../../.gitbook/assets/image (7).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (8).png" alt=""><figcaption></figcaption></figure>

It is an array of 65536 elements. At first, it seemed like all elements were 0x00, but when I copied it I started finding 0x01, 0x02,...0x09 indices:

<figure><img src="../../.gitbook/assets/image (9).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (10).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (11).png" alt=""><figcaption></figcaption></figure>

Now, let's start with condition when i=1. a\*256+b must be equal to 53937 while a and b are in (0,255)

To solve this I used z3 theorem prover in python and found the solution:

<figure><img src="../../.gitbook/assets/image (12).png" alt=""><figcaption></figcaption></figure>

I automated this using a python script and found all the solutions:

```python
from z3 import *

a,b = Ints("a b")
s = Solver()
s.add(a<255)
s.add(a>0)
s.add(b<255)
s.add(b>0)

number_dict = {
    1: 53937,
    2: 28079,
    3: 41772,
    4: 35141,
    5: 16139,
    6: 17874,
    7: 54902,
    8: 52463,
    9: 15346
}

for key, number in number_dict.items():
        s.push()
        s.add(a*256 + b == number)
        if s.check() == sat:
                print(f"For {key} = {number}, a and b are:", s.model())
        else:
                print(f"No solution for {key} = {number}")
        s.pop()
```



<figure><img src="../../.gitbook/assets/image (14).png" alt=""><figcaption></figcaption></figure>

Great! We have all the answers now. I created a small python script to help input all this data in the binary:

<figure><img src="../../.gitbook/assets/image (15).png" alt=""><figcaption></figcaption></figure>

Let's get the flag!

<figure><img src="../../.gitbook/assets/image (16).png" alt=""><figcaption></figcaption></figure>

Pretty nifty tool, z3 right? Here is a brief on theorem solvers: [https://hexisanoob.gitbook.io/hexisanoob/enum-and-initial-compromise/buffer-overflow-prep/theorem-proving](https://hexisanoob.gitbook.io/hexisanoob/enum-and-initial-compromise/buffer-overflow-prep/theorem-proving)
