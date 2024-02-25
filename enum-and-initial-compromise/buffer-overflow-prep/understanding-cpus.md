# Understanding CPUs

While doing buffer overflows we have to understand CPU functionality. CPU instructions visible in any debugger are in assembly. We talked about various pointers in the parent page. Let's focus on understandings instructions. Before that, one quick interesting fact about pointers.

### Pointers

**IP = instruction Pointer**

What should the CPU execute? ->Determined by IP

Fetches the instruction at the address in IP->Decode it->Run it

But it is more complicated than that in modern CPUs. Infact, it is not just going to the current instruction to be executed, but it will try to execute like the next 10 instructions all at once. It's going to do weird things like if it encounters an if statement, it will guess which way the if statement will go and execute along that side for a while.

And then if it turns out it was wrong, it will just roll back the effects. It is called "speculative execution"

CPU developers have tried to hide this complexity and present to programmers in a way that they feel it is just one instruction then the next and the next and so on until CPU is turned off.



### Instructions

Example of how instructions look in assembly

**mov rax, 0xdeadbeef**

In the format:

**Operation    Register(Destination)     Immediate(Source)**

\----

**mov rax, \[0xdeadbeef + rbx\*4]**

This kind of notation is meant for doing things like array access. Compilers will use them this way too that's why instructions of this format is helpful.

eg:&#x20;

int foo\[128];

Let's say \&foo (memory address where foo starts) is at 0xdeedadbeef

Then we can find foo\[i] by doing 0xdeadbeef + i\*4

This instruction could help you speculate the working:

1. This is some array at 0xdeadbeef memory here
2. rbx is currently holding the index into that array where each element is 4 bytes long

\----

**Step-Through example of assembly**

1. Initialize eax with 0xdeadbeef. This instruction is at memory location 0x804000. When CPU executes it, rip changes (as this is the current instruction so it will point to memory address 0x804000)

<figure><img src="../../.gitbook/assets/image (3) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

2. Now rip changes to the next instruction. rax gets initialized with memory address 0xdeadbeef. Next instruction is at 0x0804005, which means mov instruction must have been 5 bytes long.

<figure><img src="../../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

3. After the last operation rbx gets initialized with 0x1234. Now rip is at the next instruction 0x80400a

<figure><img src="../../.gitbook/assets/image (2) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

4. rip executes instruction at 0x0800a. rbx gets added in rax. rip is jumped.

<figure><img src="../../.gitbook/assets/image (3) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

5. rip now reaches 0x080400d.

<figure><img src="../../.gitbook/assets/image (4) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

6. rbx was incremented by 1 and becomes 0x1235. rip now jumps to the next instruction at 0x0804010

<figure><img src="../../.gitbook/assets/image (5) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

7. a subtract option is encountered. rax=rax-rbx is done.

<figure><img src="../../.gitbook/assets/image (6) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

8. rax is updated to 0xdeadbee. Then rip is jumped to 0x0804013

<figure><img src="../../.gitbook/assets/image (7) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

9. rax is initialized with rbx's value finally!

\---

## Control Flow

The instructions we saw above are good but we can't do much of high performance computing unless we start doing branches and loops and if statements and switch statements etc.  These conditionals in x86 are represented as following:

\-> Conditional jumps

* jnz \<address>
* je \<address>
* jge \<address>
* jle \<address>
* etc.

All the higher level program construct like if, for, while, switch etc end up being some variant of jump instruction.

They will jump if their condition is true, or just go to the next instruction otherwise.

But what conditions are they checking? Answer is: EFLAGS registers

\---

## EFLAGS

They store a bunch of flags about what the current state of the CPU is and what the result of most recent operations were.

For example:

**add rax,rbx**

This instruction will set the o flag if sum is greater than a 64 bit register can hold and wraps around

\->You can start a jump based on overflow eith a "jo" instruction



Another example:

**cmp rax,rbx**

1. **jle**
2. je or jz
3. jge
4. jl
5. jg

This will jump if 1. rax<=rbx, 2. rax=rbx, 3. rax>=rbx, 4. rax\<rbx, 5. rax>rbx

Under the hood cmp subtracts rbx from rax but it isn't modifying rax this time rather storing an eflag which jump conditions can use to do something!

[https://en.wikipedia.org/wiki/FLAGS\_register](https://en.wikipedia.org/wiki/FLAGS\_register)

\---

## Memory

Everything- instructions, numbers, strings is represented in hex bytes.

<figure><img src="../../.gitbook/assets/image (205).png" alt=""><figcaption></figcaption></figure>

\-> null byte indicates end of string

## Stack

<figure><img src="../../.gitbook/assets/image (206).png" alt=""><figcaption></figcaption></figure>



CPU doesn't care where the stack is. All stack is basically a designated part of memory that we understand as a stack. When we use instructions like push and pop, they are going to implicitly take the value in RSP and use that as a location of where they ar doing these operations on memory. The picture is a view of what memory looks like.

People avoid putting anything at address 0x000000000 because in programming, 0 is NULL. Let's say a program compares something and it returns 0x0. Rather than throwing exception, CPU will starting doing whatever is put at address 0x000000000.



Stack follows LIFO

Push -> on top of stack

Pop -> remove from top of stack

rsp -> points to the top of the stack

push rax -> ends up decreasing rsp by 8 (64 bit register)

* This would make: sub rsp, 8\
  &#x20;                             mov \[rsp], rax

pop rax -> increase rsp

&#x20;                                 mov rax, \[rsp]

&#x20;                                 add rsp, 8

## Little and Big endian

0xdeadbeef

Little endian: ef be ad de

Big endian: de ad be ef



## Functions

Nothing more than a bunch of operations. Usually it does stuff with the stack to create a stack frame

push rbp

mov rbp, rsp

sub rsp, 0x100



Here, a function is pushing rbp on stack. Then rsp is made to point to the top of the stack which is currently only rbp's 8 bytes.

Then finally rsp is given 100 bytes of storage. This could mean that code has some local variables that will use 100 bytes.

Why are we using rbp instead of rsp?

Let's say there are 3 variables: x,y,z.

Stack will look something like this:

<figure><img src="../../.gitbook/assets/image (208).png" alt=""><figcaption></figcaption></figure>

Here, we can see 3 variables assigned. These variables are denominated by memory addresses w.r.t rbp

Since rbp is pushed on to the stack where stack begins, it will remain constant throughout the execution of the program.

If we used rsp for variables, it will keep changing throughout the program!

Let's say we also passed some function parameters a,b,c,d,e,f, g and h to be used before we even started execution of the program, stack will look like this now:

<figure><img src="../../.gitbook/assets/image (209).png" alt=""><figcaption></figcaption></figure>



And meanwhile since after executing a function, we need to return back to the bottom of the stack (with ESP still high) to see what to execute next, we add return address on rbp+8

<figure><img src="../../.gitbook/assets/image (210).png" alt=""><figcaption></figcaption></figure>

When a function exits, control goes back to return address leaving all the memory in tact.



According to SystemV AMD64 ABI, the first 6 arguments to a function are passed from left to right in these registers:

rdi,rsi,rdx,rcx,r8,r9

Further arguments are pushed to the stack

The return value of the function is stored in rax when the function returns



<figure><img src="../../.gitbook/assets/image (211).png" alt=""><figcaption></figcaption></figure>



