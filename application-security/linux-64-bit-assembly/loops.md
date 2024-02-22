# Loops

In Assembly, a loop decrements ECX register. When ECX==0, loop will end.

We can make a loop using either control instructions like jmp, jnz etc (like in last page) or we can also use the loop instruction as well!

Make sure we preserve ecx throughout the program. Use stack to do this.

Question: Write a loop to display "hello world" 10 times using a loop.



```
; Self-written program for loops

global _start

section .text

_start:
        xor rax,rax
        mov rax,10
Display:
        push rax ; storing rax's state
        mov rax, 1 ; 1 for syscall write
        mov rdi, 1 ; 1 for int fd=1 to stdout
        mov rsi, message ; message hello world
        mov rdx, len ;  length of the message
        syscall ; Calling to print message

        ;Now loop should be decremented until 10 times (until it becomes 0)
        pop rax
        dec rax
        jnz Display

Ending:
        ;program should exit safely
        mov rax,60
        mov rdi,0 ; error code is 0
        syscall

section .data

        message: db 'Hello World',0xa
        len equ $-message
```



<figure><img src="../../.gitbook/assets/image (359).png" alt=""><figcaption></figcaption></figure>

As we can see here, I am using the rax technique as in last article to loop. Let us use the instruction "loop" to re-write this:

```
; Self-written program for loops

global _start

section .text

_start:
        xor rcx,rcx
        mov rcx,10
Display:
        push rcx ; storing rcx's state
        mov rax, 1 ; 1 for syscall write
        mov rdi, 1 ; 1 for int fd=1 to stdout
        mov rsi, message ; message hello world
        mov rdx, len ;  length of the message
        syscall ; Calling to print message

        ;Now loop should be decremented until 10 times (until it becomes 0)
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

<figure><img src="../../.gitbook/assets/image (360).png" alt=""><figcaption></figcaption></figure>

Using loop instruction, we are utilizing rcx, avoiding the risk of spoiling rax. We are also reducing the hassle of decrementing rax on loops. Finally, we are simplifying some logic too.



Please note, if we don't preserve rcx here, after the write syscall, rcx will become all f's. This would become an infinite loop. So we have to preserve this value manually using a stack.





There are some variations of loop as well: LOOPE, LOOPNE, LOOPNZ, LOOPZ Ref Page 652 of the intel manual ([https://www.intel.com/content/dam/www/public/us/en/documents/manuals/64-ia-32-architectures-software-developer-vol-3c-part-3-manual.pdf](https://www.intel.com/content/dam/www/public/us/en/documents/manuals/64-ia-32-architectures-software-developer-vol-3c-part-3-manual.pdf))
