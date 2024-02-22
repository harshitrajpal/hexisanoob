# Arithmetic Operations

Use of RFLAGS register is done majorly

We look at these main important operations:

1. ADD destination, source
2. ADC destination, source -> Add with carry
3. SUB
4. SBB -> Subtract with borrow
5. INC -> Increment
6. DEC -> Decrement

```
global _start

section .text
_start:

        ; register based addition 

        mov rax, 0x01
        add rax, 0x01 ; Adds 1 to rax

        mov rax, 0xffffffffffffffff
        add rax, 0x01 ; Adds 1 to RAX

        mov rax, 0x09
        sub rax, 0x04 ; Subtracts 4 from RAX


        ; memory based addition 

        mov rax, qword [var1]
        add qword [var4], rax

        add qword [var4], 0x02


        ;  set / clear / complement carry flag 

        clc
        stc
        cmc

        ; add with carry 

        mov rax, 0
        stc
        adc rax, 0x1
        stc
        adc rax, 0x2

        ; subtract  with borrow

        mov rax, 0x10
        sub rax, 0x5
        stc
        sbb rax, 0x4

        ; increment and decrement 

        inc rax
        dec rax

        ; exit the program gracefully  

        mov rax, 0x3c
        mov rdi, 0
        syscall

section .data

        var1    dq      0x1
        var2    dq      0x1122334455667788
        var3    dq      0xffffffffffffffff
        var4    dq      0x0
```

We can assemble and link this to make an executable and use GDB in TUI mode to see how arithmetic operations are happening.

I'm attaching the screenshots and writing a short description of the programs.

<figure><img src="../../.gitbook/assets/image (319).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (320).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (321).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (322).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (323).png" alt=""><figcaption></figcaption></figure>

In the screenshots above, I'm first adding 1 to RAX. Then adding 1 again. Then moving fffffffffffff to RAX overwriting 2. Then adding 1 to make it 0. Also note how Carry Flag (CF) becomes 1 (see RFLAGS register)

<figure><img src="../../.gitbook/assets/image (324).png" alt=""><figcaption></figcaption></figure>

&#x20;Then adding 1 and 9 and subtracting 4 finally depicting add and subtract operations.

At this point in time, rax is assignded as the value in address 0x402000. Upon seeing the variables we see it is var1.

<figure><img src="../../.gitbook/assets/image (325).png" alt=""><figcaption></figcaption></figure>

This variable has value 1.

<figure><img src="../../.gitbook/assets/image (326).png" alt=""><figcaption></figcaption></figure>

Now we are adding rax's value in the variable stored at 0x402018

<figure><img src="../../.gitbook/assets/image (327).png" alt=""><figcaption></figcaption></figure>

Then we are adding 2 in this.

Next instruction is clc -> Which clears the carry flag.

stc-> sets the carry flag

<figure><img src="../../.gitbook/assets/image (328).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (329).png" alt=""><figcaption></figcaption></figure>

CMC -> complement the carry flag. CF was 1 just now, upon cmc, it gets 0 again.

Next three instructions are:\
mov rax, 0x0\
stc\
adc rax,0x1

This basically means that first rax becomes 0. Then carry flag is set (becomes 1) and then adc-> Add with carry.

ADC=RAX+\<value provided>+CF value

Here, value provided is 0x1, CF is also 1, so RAX should become 2.

<figure><img src="../../.gitbook/assets/image (330).png" alt=""><figcaption></figcaption></figure>

After adding, notice how CF again becomes 0. Next instructions set it again and adds with carry rax, value and CF's value which becomes 5.

next, we mov value 0x10 to rax then subtract 5 from rax making it 0xb (10 in hex is 16 in decimal. So, 16-5 = 11 which is 0xb)

<figure><img src="../../.gitbook/assets/image (331).png" alt=""><figcaption></figcaption></figure>

Now, we have 0xb = 11 in RAX. We set carry flag. Then we use SBB-> Subtract with Borrow.

So, SBB= RAX - Value Provided - Carry Flag

So, SBB makes RAX=11-4-1 = 6

<figure><img src="../../.gitbook/assets/image (332).png" alt=""><figcaption></figcaption></figure>

Finally, INC and DEC just increments the register mentioned (RAX) by 1 and decreases by 1.

Then we exit the program

