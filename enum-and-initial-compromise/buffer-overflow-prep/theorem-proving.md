# Theorem Proving

Some programs are easy to bruteforce and get an answer to something(like in CTF challenges). But some programs are more complicated in nature and they often won't make it easy to bruteforce them like that. For example, a program sleeps for 5 seconds after supplying input. So bruteforcing becomes hard.

One way out is to open ghidra, copy the decompiled code in a file, compile it and bruteforce it that way.

Other way is by using a theorem prover!

## Theorem Proveer

Theorem proving can help solve complex problems with "simple" input. General concept is that given a set of contraints, a theorem prover will find a solution to satisfy all of them or tell you if it's not satisfiable.

Most common and robust theorem prover is z3: [https://github.com/Z3Prover/z3](https://github.com/Z3Prover/z3)

z3 supports many types. Most commonly:

* Ints (arbitrary length integer)
* BitVecs (integers of a specific bit length)
* Bools (T/F)
* Solver (how we check for output from the engine)



**pip3 install z3-solver**

```
In [1]: from z3 import Ints, Solver

In [2]: a, b = Ints("a b")

In [3]: a
Out[3]: a

In [4]: b
Out[4]: b

In [5]: s=Solver()

In [6]: s.add((a+b)*2==100)

In [7]: s.add((a-b)*10 == 0)

In [8]: s.check()
Out[8]: sat

In [9]: s.model
Out[9]: <bound method Solver.model of [(a + b)*2 == 100, (a - b)*10 == 0]>

In [10]: s.model()
Out[10]: [b = 25, a = 25]
```

<figure><img src="../../.gitbook/assets/image (17).png" alt=""><figcaption></figcaption></figure>

Solver.add() => Adds conditions to the model

Solver.sat() => Checks if the conditions added satisfy a solution or not. sat=satisfiable or unsat=unsatisfiable

Solver.model() => Outputs the solution



Another example for the same equations we provided:

<figure><img src="../../.gitbook/assets/image (1) (1).png" alt=""><figcaption></figcaption></figure>



Now, here is a simple fibonacci recursive function in C. Find the value of a and b which would return 1 upon satisfying the first if condition:

```clike
int recurse(int a, int b, int c) {
  int sum = a + b;
  if (c == 16 && sum == 116369) {
    return 1;
  } else if (c < 16) {
    return recurse(b, sum, c + 1);
  } else {
    return 0;
  }
}=
```



As we can see here:

when c=1, sum = a+b

Then recursive function is faced and then:

a becomes b

b becomes sum(a+b)

So, lets say at c=1, a=1, b=1; sum=2

c=2: a=b;b=a+b => a=1;b=2; sum=3

c=3: a=a;b=a+b => a=2;b=3;sum=5

...and so on



The general transformation for a and b is: a=b, b=a+b

This goes on until c becomes 16.

So, python function becomes:

```
def recurse(a,b):
    for i in range(1,17):
        a=b
        b=a+b
    return b
```



Using z3 this can be solved like:

<figure><img src="../../.gitbook/assets/image (2) (1).png" alt=""><figcaption></figcaption></figure>



<figure><img src="../../.gitbook/assets/image (3) (1).png" alt=""><figcaption></figcaption></figure>

So, at a=13 and b=37, we'll have c=16 and sum=116369

## Symbolic Execution

Rather than treating input as a concrete value, a symbolic execution refers to putting in a symbolic value (variable) and running through a program keeping track of what operations were done on the symbolic variable and trying to explore all possible paths on the program.

<figure><img src="../../.gitbook/assets/image (4) (1).png" alt=""><figcaption></figcaption></figure>

In the screenshot above, when z==12, fail() is called and when z!=12, OK is printed. So, a variable lambda (λ) is read and assigned to y.

λ could take any value, and symbolic execution forks two paths, as it can proceed along both branches.

So, the constraint solver would determine that in order to reach fail(), λ would be equal to 6.



But as conditionals increase, each conditional creates two new branches of possibility. So for a large program, exponential paths will be created. This is not feasible. Its recommended to restrict symbolic execution to only a small part of the program which is relevant.



## Automated symbolic execution on binaries

When doing this on binaries, models like z3 don't work as well as expected because for these, we need to take symbolic data and run it through assembly code.

We can use angr here!









