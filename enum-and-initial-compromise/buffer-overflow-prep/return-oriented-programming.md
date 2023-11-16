# Return Oriented Programming

In most challenges so far, NX was disableed, or memory address of a function to jump to was there or system() was there to use.

What to do in the case we don't have these assistances?

That's when ROP comes in to play

ROP is a code resuse attack - We don't need our own shellcode (but we can if we want). We modify the control flow in a way that executes what we want, using nothing but simple "ret" priimitives.

Concept at a high-level:&#x20;

○ you have smashed the stack, but it’s non-executable&#x20;

○ there is no address you can jump to in order to pop a shell&#x20;

○ but, you can still ret somewhere and use the stack data to do other things (i.e., pop)&#x20;

○ ...and you can ret again after that&#x20;

○ …...and after that&#x20;

○ ……..and after that



Reference: [https://hovav.net/ucsd/dist/geometry.pdf](https://hovav.net/ucsd/dist/geometry.pdf)

\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_

How often does a program contain a sequence of instructions that may help in a shell? Like system() instruction somewhere.

Functions aren’t really a “thing” in x86…&#x20;

○ You can jump to the middle of a function, and that’s all fine! \
● Instructions are variable-length in x86…&#x20;

○ From 1 to 15 bytes for an instruction&#x20;

○ You can jump into the middle of an instruction, and that’s fine!&#x20;

○ Jumping at different points yields different opcodes, which means we have a lot more instructions to use than the programmer intended&#x20;

● If we can smash the callstack, we can control return pointers&#x20;

● That means we can do evil things like return, then return again, etc...

\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_

x86 only _kind of_ understands functions. What it really understands is the "call stack"

\->call and ret instructions push and pop to the callstack, for instance&#x20;

● What does the ret instruction really do?

\->pop rip&#x20;

○ If we control the stack, we control what it pops. That means we can control multiple returns, each one making small changes

\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_

For example:

An instruction is:

mov rax, 0xc30f05 ; ret == 48 c7 c0 05 0f c3 00 c3

Which is moving an address in RAX and then returning to an address.

If we jump IP to just 05 0f c3, we can get a "syscall ; ret"



A program that helps finding these little gadgets: [https://github.com/JonathanSalwan/ROPgadget](https://github.com/JonathanSalwan/ROPgadget)



\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_

IMportant thing to note here is that when using POP, essentially the value on the stack currently is assigned to the register told by POP.&#x20;

Let me explain this with an example. I want to make a program execute: execve(“/bin/sh”, NULL, NULL).

execve is the 59th system call. Which is 0x3b in hex.

Assuming the string "/bin/sh" is available at address 0x600000

Can we make a shellcode only using ROPgadgets to make the program execute a shell?

<figure><img src="../../.gitbook/assets/image (263).png" alt=""><figcaption></figcaption></figure>

Well, from the website: [https://filippo.io/linux-syscall-table/](https://filippo.io/linux-syscall-table/) for execve, first argument we need to set rdi to the 0x600000 location. Then we need to set rsi and rdx to NULL (0x0) and finally we need to set RAX to 0x3b (system call execve's number in hex). We know that all system calls are executed using syscall assembly function. At that time, RAX should be 0x3b to execute execve.



Now, interesting thing about POP is that it basically sets a register.&#x20;

* "pop" removes the last value pushed onto the stack, and stores it in a register.
*   For example, this loads 3 into rax and returns.  It's a kinda roundabout way to return a 3, but between the push and the pop you could use rax for something else until you need it.\


    ```
    push 3
    pop rax
    ret
    ```

Reference: [https://www.cs.uaf.edu/2017/fall/cs301/lecture/09\_08\_stack.html](https://www.cs.uaf.edu/2017/fall/cs301/lecture/09\_08\_stack.html)

So essentially by lining up Gadgets with addresses to different pop instructions we can make our own ROP shellcode.

In  the question above, need these 6 instructions in this sequence to make our shellcode

A. pop rdi; ret -> "/bin/sh" string's memory location

B. pop rsi; ret ->2nd NULL argument of execve

C. pop rdx;ret -> 3rd NULL argument of execve

D. pop rax;ret -> number assigned here would tell system which syscall. 0x3b is execve

E. syscall; ret -> location of syscall in memory somewhere

* where A,B,C,D,E are memory addresses of these instructions

Then since we know that POP will assign a register with a value on top of the stack, we can craft a payload.

Let's say EIP offset is 42.

Payload would become:

**42\*A's + A + 0x600000 + B + 0x0 + C + 0x0 + D + 0x3b + E**

And this is how we will essentially create a payload that would execute this command: execve(“/bin/sh”, NULL, NULL)

Assembly would be:

```
mov rdi, 0x600000 
mov rsi, 0 
mov rdx, 0 
mov rax, 0x3b 
syscall
```



\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_

Again, this begs the doubt-- what if these instructions are not in my binary?

A binary could be small with not many gadgets to play around with. What to do in such case?

As we know that all binaries have linked libraries ( explained in the linking - GOT,PLT page in this section). And these libraries are huge. They have loads and loads of gadgets. One can essentially use these gadgets in their shellcode.



