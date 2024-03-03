# push, pop, and the stack

We already know how a stack works. Here is a previous article to refer to: [https://hexisanoob.gitbook.io/hexisanoob/enum-and-initial-compromise/buffer-overflow-prep](https://hexisanoob.gitbook.io/hexisanoob/enum-and-initial-compromise/buffer-overflow-prep)



When we push on stack, the RSP increments from a higher memory address to lower. When we pop, the RSP decrements from lower to higher memory address.

Here is a code I wrote in assembly that performs basic push and pop opperations

```
global start

section .text
_start:

        mov rax,0xaaaaaaaabbbbbbbb
        push rax ; This would put rax's value on the top of stack

        push var1 ; This puts var1 on the top of the stack

        push qword [var1] ; This is a typecasting example. This typecasts the sequence of bytes in var1 as a quadword (8bytes) and puts on stack

        pop r15 ; pops r15 and so on for next 3 instructions
        pop r14
        pop rbx

        ; Exiting the program
        mov rax, 0x3c
        mov rdi, 0
        syscall


section .data
        var1: db 0xaa, 0xbb, 0xcc, 0xdd, 0xee, 0xff, 0x11, 0x22
```

Let's launch this code in GDB TUI mode and layout regs and asm. We'll step through and dissect this code to see what is happening to the registers as we step to the next instructions.

<figure><img src="../.gitbook/assets/image (2) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Now, we can't see the stack as of now. So we'll set up a hook stop. This is a special command which is executed after every step: [https://sourceware.org/gdb/current/onlinedocs/gdb.html/Hooks.html](https://sourceware.org/gdb/current/onlinedocs/gdb.html/Hooks.html)

I'm setting up a hook-stop to examine 4 giant words at the top of the stack (RSP)

**define hook-stop**\
**x/4xg $rsp**\
**end**

As we can see, the stack is now visible after every next step we take in GDB.

<figure><img src="../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

We take two next steps and observe how stack is now updated

<figure><img src="../.gitbook/assets/image (2) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

We can similarly see how pop is working now. The value on top of the stack is now popped and stored in r15.

<figure><img src="../.gitbook/assets/image (3) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Similarly, other pops are working too

<figure><img src="../.gitbook/assets/image (4) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Finally, the program exits.

Through this demo we are seeing how push and pop works. How stack is being populated nad we are visually seeing the stack as we go. Then we can also pop certain values and put them in different registers. This is another way to move data btw! This would be helpful in ROP and ROP-buffer overflows

