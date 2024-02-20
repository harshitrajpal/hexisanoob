---
description: >-
  Provides a simple hello world program with  comments that explain what is
  happening
---

# Basic Hello World Program

Refer to the syscall tables here:  [https://filippo.io/linux-syscall-table/](https://filippo.io/linux-syscall-table/)

Also at: /usr/include/x86-64-linux-gnu/unistd\_64.h file in linux

Basically there are 3 program  sections in  assembly:

.data  section:  Which holds the  variables and strings

.text section: Holds the CPU instructions in Assembly

\_start variable: This is where CPU execution begins and RIP is assigneed to  the start of this offset when program executes.

We use general  purpose registers for syscalls. Like rax and assign a value to it and use rdi,rsi,rdx,r10,r8,r9 as the 1st,2nd,3rd...6th argument  of the syscall.



Upon a syscall execution, the return  value of that syscall is also stored in rax again.

Considering this basic, here is a very simply hello world program which would  write to screen and then exit.

```
global start

section .text

        _start:
        mov rax,1 ;syscall for write.
        mov rdi,1 ;int fd, which is 1 for stdout since we are not storign the  message anywhere rather printing it out
        mov rsi,hello_world ; The  buffer to be printed
        mov rdx,length ; The length of   hello world buffer we calculated in data section
        syscall ; Executes the syscall 1 with arguments we gave. At this point rax  will store the return value from the executed syscall
        mov rax, 60 ; syscall number for exit
        mov rdi, 11 ; Custom error code. Could be anything. 0 in most cases.
        syscall

section .data
        hello_world: db "Hello world! This is Harshit"
        length: equ $-hello_world
```

To assemble and link and finally execute this:

nasm -f elf64 hello.asm -o hello.asm

ld hello.o -o hello

./hello

<figure><img src="../../.gitbook/assets/image (300).png" alt=""><figcaption></figcaption></figure>

Here, in .data section "db" is short for define bytes which initializes a string.  We can input enters in assembly by adding 0a at the end.

P.S.  Semicolons in assembly are comments

<figure><img src="../../.gitbook/assets/image (301).png" alt=""><figcaption></figcaption></figure>

