# Control Instructions

Alter the flow of the programs based on different events. Eg: If a calculation has happened and the result is 0. So, for example, JZ could be used to make it JMP somewhere.



Conditional jumps: JA,JE,HNC,JNZ etc -> Uses flags

Unconditional jumps: JMP -> Equivalent to GOTO in C.

Demo:

```
global _start

section .text
_start:

        jmp Begin

NeverExecute:

        mov rax, 0x10
        xor rbx, rbx

Begin:
        mov rax, 0x5

PrintHW:

        push rax

        ; print on screen

        mov rax, 1
        mov rdi, 1
        mov rsi, message 
        mov rdx, mlen
        syscall

        pop rax
        dec rax
        jnz PrintHW


        ; exit

        mov rax, 60
        mov rdi, 11
        syscall

section .data

        message: db "Hello World! ", 0x0a
        mlen     equ $-message
```



Dissecting this we can understand the behavior

<figure><img src="../../.gitbook/assets/image (351).png" alt=""><figcaption></figcaption></figure>

First, we use JMP to Begin label. We put 0x5 on eax and push rax on the stack. Again using hook stop on $rsp to view stack as we go

<figure><img src="../../.gitbook/assets/image (352).png" alt=""><figcaption></figcaption></figure>

Here, we are now trying to call write syscall with our message in rsi and length in rdx with fd=1 for stdout in rdi (edi, rdi is equivalent here and assembler has optimized the registers for memory)

<figure><img src="../../.gitbook/assets/image (353).png" alt=""><figcaption></figcaption></figure>

The message is printed now as you can see.

<figure><img src="../../.gitbook/assets/image (354).png" alt=""><figcaption></figcaption></figure>

But really the program is a small loop that will print the message "Hello World!" 5 times.&#x20;

<figure><img src="../../.gitbook/assets/image (355).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (356).png" alt=""><figcaption></figcaption></figure>

Now, original value 5 is put on rax again using pop. Then decremented by 1 to make it 4. Then JNE is encountered which sends the control back to PrintHW label where push rax stores 4 on stack now. It becomes a loop to print hello world

<figure><img src="../../.gitbook/assets/image (357).png" alt=""><figcaption></figcaption></figure>



This is happening because we are utilizing the stack to store current rax value. Because the write operation is changing rax value, we are using stack to maintain state of  RAX. It decrements 5 times and then when it becomes 0, jne statement becomes false and the program exits. MAGIC!

<figure><img src="../../.gitbook/assets/image (358).png" alt=""><figcaption></figcaption></figure>



