# Linking - GOT,PLT

Write What Where vulnerability -> Technically an exploit primitive (as in some ability that the program gives you as a result of another vulnerability) You can write whatever you want, wherever you want. GOT has RW privileges. We can overwrite a GOT entry to execute whatever we want to be executed the next time that GOT entry is called

\-----------------------------------------------------------------------------------------------------

When we run a program what is the first thing that runs? main? NO.

There is a bunch of stuff that needs to be setup before running the main function like the header files, global variables and such. This could be found by setting a break point at \_start function. This is the first function that executes and sets up the environment.

<figure><img src="../../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

In stripped binary with symbols removed, decompiler like ghidra won't know where main is. It will start off at this \_start function and then we have to make sense of it and figure out where main is

### Static vs Dynamic Linking

A program requires a lot of functionality that you probably don’t want to code yourself. E.g., fopen, gets, print, puts, etc. To implement these functions without having to write them all the time manually, we have these functions in a dynamic library called "libc." Then there are common 3rd party libraries that are commonly used like Boost, openssl, etc. which are in libraries like libssl. If you want to use these libraries, you can either dynamically or statically link



We are taking example of printf() throughout the article to understand concepts.



Essentially, whenever we call a function like printf() we have to know it's address. So how would that work if the related code to that function is in some other library? So we need something that would translate this function printf() in Assembly to a memory address of the function description in the respective library and makes a call to that function. This is where linker comes in.



STATIC Linking:

* Everything is brought into the binary
* The binary doesn’t need anything else from the system
* The binary is going to be quite large

DYNAMIC Linking:

* Use the libraries that are (likely) already present on the system
* Binary is quite small and simple, as well as much faster to build

All the linked libraries in a binary can be seen using ldd command

<figure><img src="../../.gitbook/assets/image (2) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Here, libc.so.6 is the C standard library where printf lives. ld-linux-x86-64.so.2 is the runtime loader. It decides where the program should go in memory and how to resolve the name printf() in the address of the printf() function in libc. linux-vdso.so.1 is a Linux feature which allows certain system calls to be implemented without having to jump into the kernel. It improves performance so it is linked by default.

This is an example of dynamic linking. By default GCC dynamically linked my program.

But now when we run the program an extra step has to happen where the functions used in the code have to get turned into the addresses in the libraries they are implemented in.

### Dynamic Linking

Libraries basically export a list of what functions they provide. It's called "export" and is a big list of address and name. These exports are stored in Global Offset Table (GOT).&#x20;

Some steps are taken to dynamically link:

1. When a program is first loaded in the memory, the address of printf is set to a little chunk of code in the program itself. It's not the full implementation. It's called a stub. Stub invokes the linker and asks the address of printf.
2. The linker loads the necessary library (libc) into the process memory space.
3. The linker resolves the function address of printf
4. Stub is now replaced with actual printf's memory address into the GOT.

We can think of the GOT as a bunch of trampolines that are resolved (bounced to right location) as needed.

Ghidra->window->memory map

<figure><img src="../../.gitbook/assets/image (3) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (4) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

In ghidra, we see a PLT (procedure linking table) for some function in binary. The stub is visible in the 2nd screenshot. It is just a jump instruction.



**In PLT, the stub lives. It looks up the pointer value in GOT and then jumps to it.**



Let's disassemble and see in our hello binary where printf lives.

**objdump -d hello -M intel**

<figure><img src="../../.gitbook/assets/image (5) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

In main() where printf is, we see something written after printf. It is printf@plt which stands for procedure linkage table. Let's inspect the printf@plt function in objdump

<figure><img src="../../.gitbook/assets/image (6) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

We can see that printf is just shown as a 3 instruction function while in reality it is much bigger than that. This printf@plt is a stub which is linking to the real printf function in GLIBC\_2.2.5

In GDB, we can examine memory address for printf@plt we will get memory address for real printf. We can disas \<memory address> to see printf's assembly implementation.



### Abusing Dynamic Linking

Since GOT gets resolved in runtime (initially all pointers are null/garbage value. Then linker updates it's values), it has to have RW permission. So if we have a vulnerability in program that lets you write in memory, we can write in GOT because updating an address here would essentially replace one of the library functions with a function of our own.

So, if print's entry in GOT is changed, the next time printf will be called, it will be replacecd with our code and instead our function will be executed.

Let's see an example



Demo:

I created a simple binary that would execute a system command date. In main I simply input a buffer and nothing else.

Goal: make system() execute whatever buffer is entered by replacing puts' GOT entry with system() GOT entry.



<figure><img src="../../.gitbook/assets/image (12).png" alt=""><figcaption></figcaption></figure>



In objdump, we can see puts@plt and then other symbols as well. The system function also exists and as we can see it is linked with the related library as well.

<figure><img src="../../.gitbook/assets/image (11) (1) (1).png" alt=""><figcaption></figcaption></figure>

Now, we will set two breakpoints. One in main. Then run the program. Then set a breakpoint just before the second puts is called.

<figure><img src="../../.gitbook/assets/image (13).png" alt=""><figcaption></figcaption></figure>

Now, when this break point is hit after continuing, let's observe the GOT mappings using x/xg

<figure><img src="../../.gitbook/assets/image (14).png" alt=""><figcaption></figcaption></figure>

GOT mappings for puts and fgets are visible here. Let's see for system as well. These hex values represent GOT entries (just pointers!) With info symbol command we can see which GOT entry belongs to which function. Here, puts in section .text.... signifies that puts() is linked with the library libc

<figure><img src="../../.gitbook/assets/image (16).png" alt=""><figcaption></figcaption></figure>

Let's try to manually change this puts mapping with system mapping

<figure><img src="../../.gitbook/assets/image (17).png" alt=""><figcaption></figcaption></figure>

So now, system will be called with puts() arguments and whatever buffer I pass will be executed. What's a fun command to run? Hmm. Let's see /etc/passwd! Let's start the process again in one smooth flow.

1. set main breakpoint
2. set breakpoint just before the second puts

<figure><img src="../../.gitbook/assets/image (18).png" alt=""><figcaption></figcaption></figure>

3. When breakpoint is hit, examine GOT entries. Replace puts@got.plt with system@got.plt
4. Continue execution to see system() executed with the buffer input in fgets which was supposed to be executed by puts but is now executed by system()

breakpoint is hit. Let's set puts entry to system's entry as we saw above and continue execution!

<figure><img src="../../.gitbook/assets/image (19).png" alt=""><figcaption></figcaption></figure>

We have successfully changed GOT entry and made the binary execute a system command.



\-----------------------------------------------------------------------------------------------------

This example simulated write what where vulnerability in GDB

In short,

printf@plt in disassembly of main -> stub (jumper function in PLT which is pointing to a value in got.plt) -> printf@got.plt on some address -> info symbol \<that address> -> linker info of the real printf function

\-----------------------------------------------------------------------------------------------------

Mitigations of Dynamic Linking:

1. RELRO: Relocations Read-Only -> Full and Partial

Partial RELRO does nothing against this exploit

Full RELRO would resolve all the pointers before execution

\-----------------------------------------------------------------------------------------------------

An example in CTF is given here: [https://hexisanoob.gitbook.io/hexisanoob/ctf-challenge-learnings/pwning/plt\_playit](https://hexisanoob.gitbook.io/hexisanoob/ctf-challenge-learnings/pwning/plt\_playit)
