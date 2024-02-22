# Procedures

Set of operations grouped together. They can be called from different places in the code.

CALL-> Instruction can be used. CALL Procedure\_Name

In NASM, procedures are defined using labels.



Eg:

ProcedureName:

..code..

..code..

RET



So, let's rewrite a simple hello world program with loops we used in last article using procedures

```
; Self-written program for loops

global _start

section .text

HelloWorldProc:

        mov rax, 1 ; 1 for syscall write
        mov rdi, 1 ; 1 for int fd=1 to stdout
        mov rsi, message ; message hello world
        mov rdx, len ;  length of the message
        syscall ; Calling to print message

        ret

_start:
        xor rcx,rcx
        mov rcx,10

Display:
        push rcx ; storing rcx's state
        call HelloWorldProc
        pop rcx
        loop Display

Ending:
        ;program should exit safely
        mov rax,60
        mov rdi,0 ; error code is 0
        syscall

section .data

        message: db 'Hello World',0xa
        len equ $-message
```

<figure><img src="../../.gitbook/assets/image (361).png" alt=""><figcaption></figcaption></figure>

How do CALL and RET perform this? Using stack. Dissect this using GDB we can seee how they work

1. Notice we are about to CALL here. RSP currently holds RCX's value that we pushed.

<figure><img src="../../.gitbook/assets/image (362).png" alt=""><figcaption></figcaption></figure>

Now, when I hit the next intruction, CALL calls our procedure, stores memory address of the next instruction (0x40102a) on the stack and procedure is called.

<figure><img src="../../.gitbook/assets/image (363).png" alt=""><figcaption></figcaption></figure>

3. Now, procedure executes but when "ret" is hit, let's see what happens. Notice how RET has popped the memory address off the stack and RIP has used this address to return back to the next instruction that was to be called after calling the procedure.

<figure><img src="../../.gitbook/assets/image (364).png" alt=""><figcaption></figcaption></figure>

4. Next instruction is POP RCX. Which would take 10 off the stack and restore RCX which is disturbed due to the system calls we made just now. It becomes 10, LOOP instruction decrements it to 9, stores on stack , procedure is called, returned.... so on until 10 Hello World appear on the screen



Why is it important?

Again, when doing buffer overflows using ROP gadgets, we need instructions that can help us control the flow of EIP. Using instructions like CALL,RET,PUSH,POP we can achieve a sequence where we can make the program run in an unintended way.



