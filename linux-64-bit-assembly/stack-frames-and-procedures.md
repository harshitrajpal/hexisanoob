# Stack-Frames and Procedures

When a procedure is called, we need to create a Stack Frame.

This Stack Frame acts like a wall and isolates all the data created by previous procedures. Typically it's created at the current location of RSP. It is removed when the procedure returns.

Why do we need it? -> If the program isn't touching the stack,  it is not needed

But according to stackovverflow.com: "Stack frames are convenient when you save registers and store local variables in stack - to make writing and debugging easier: you just set `ebp` to a fixed point in stack and address all stack data using `ebp`. And it's easier to restores `esp` at the end.

Also, debuggers often expect stack frames to be present, otherwise you can get inaccurate stack call for example." ([https://stackoverflow.com/questions/17239781/x86-assembly-why-do-i-need-stack-frames](https://stackoverflow.com/questions/17239781/x86-assembly-why-do-i-need-stack-frames))

Here is an example. When I create a function in C to print something:

<figure><img src="../.gitbook/assets/image (6) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Upon inspecting the disassembly of main we will see a stack frame set up as the first two instructions and then always ends in instructions: leave and ret.

<figure><img src="../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

We would see in many assembly programs that the instructions:

push rbp\
mov rbp,rsp\
\
Is always given at the start and then they end with

leave\
ret



Leave-> It is basically doing the reverse of push rbp;mov rbp,rsp

that is

leave=>\
mov rsp,rbp\
pop rbp



This is essentially maintaining the stack frame.

Here is an example of our previous program while maintaining the stack frame in assembly:

```
global _start
section .text

HelloWorldProc:

        push rbp
        mov rbp, rsp

        ; Print hello world using write syscall

        mov rax, 1
        mov rdi, 1
        mov rsi, hello_world 
        mov rdx, length
        syscall

        ; mov rsp, rbp
        ; pop rbp
        ; These two instructions above are  replaced with "leave"

        leave
        ret   ; signifies end of procedure

_start:

        mov rcx, 0x10

PrintHelloWorld:

        push rcx
        call HelloWorldProc
        pop rcx
        loop PrintHelloWorld

        ; exit gracefully 

        mov rax, 60
        mov rdi, 11
        syscall


section .data

        hello_world: db "Hello World!", 0x0a
        length     equ $-hello_world
```

