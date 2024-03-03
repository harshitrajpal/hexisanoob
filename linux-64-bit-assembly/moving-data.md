# Moving Data

Before going ahead, here are three rules:

a) In 64-bit mode, operands generate a 64-bit result in the destination GP register\
b) In 32-bit mode, operands generate a 32-bit result, zero-extended to a 64-bit result in the destination  general purpose register\
c) In 8-bit and 16-bit mode, operands generate an 8 or 16 bit result. The upper 56 or 48 bits of the GPR are untouched\
d) If the result of an 8 or 16-bit operation is intended for 64-bit address calculation, explicitly sign-extend the register to the full 64-bits.

"Zero extended"??? We'll  talk about this after introducing a few operands that are used to move data in assembly.

### MOV instruction

MOV is the most common instruction in assembly. It allows data moving in the following formats:

1. Between registers
2. Memory to registers and Registers to Memory
3. Immediate data  to registers
4. Immediate data to memory

### LEA

LEA=> Load Effective Address.

It loads pointer values in registers

Eg: LEA RAX, \[var1]

Where var1 is the label given to any data type (talked in data type article here: [https://hexisanoob.gitbook.io/hexisanoob/application-security/linux-64-bit-assembly/data-types](https://hexisanoob.gitbook.io/hexisanoob/application-security/linux-64-bit-assembly/data-types))



Please note that, these two instructions essentially mean the same thing:

mov rax, sample

lea rax,\[sample]



### XCHG

Swaps values in between:

1. Register and Register: XCHG RAX, RBX
2. Register and Memory: XCHG RAX, \<memory address>



Demo:

Here is an assembly program for us to dissect into.

```
global _start

section .text
_start:

        ; mov immediate data to register 
        mov rax, 0xaaaaaaaabbbbbbbb
        mov eax, 0xaaaaaaaa
        mov rax, 0xaaaaaaaabbbbbbbb
        mov al, 0x11
        mov rax, 0xaaaaaaaabbbbbbbb
        mov ah, 0xcc
        mov rax, 0xaaaaaaaabbbbbbbb
        mov ax, 0xdddd

         
        ; mov register to register 

        mov rbp, rax
        mov r10, rbp

        mov r11d, r10d
        mov r12w, r11w
        mov r13b, r12b


        ; mov from memory into register 

        mov rsi, [sample2]
        mov r14d, [sample]
        mov r15w, [sample]
        mov dil, [sample]


        ; mov from register into memory 

        mov rax, [sample2]
        mov byte [sample], al
        mov word [sample], ax
        mov dword [sample], eax
        mov qword [sample], rax


        ; lea demo

        lea rax, [sample]
        lea rbx, [rax] 


        ; xchg demo 
        mov rax, 0x1234567890abcdef
        mov rbx, 0x9999999999999999

        xchg rax, rbx
 

        ; exit the program gracefully  

        mov rax, 0x3c
        mov rdi, 0
        syscall


section .data

sample: db 0xaa, 0xbb, 0xcc, 0xdd, 0xee, 0xff, 0x11, 0x22
sample2: dq 0x1122334455667788
sample3: times 8 db 0x00
```

Note at the end we are exiting by giving  rax a value 0x3c which is hex equivalent of "60" which is the syscall number of  exit()

**gdb -q ./MovingData -tui**

Instruction 1: Rule A applies

<figure><img src="../.gitbook/assets/image (304).png" alt=""><figcaption></figcaption></figure>

Instruction 2: Note how a 32 bit value output zeros out the upper 32 bits of the register. 3rd instruction resets RAX.

<figure><img src="../.gitbook/assets/image (305).png" alt=""><figcaption></figcaption></figure>

Instruction 4:  Rule C applies and the remaining 30 bits are unaffected

<figure><img src="../.gitbook/assets/image (306).png" alt=""><figcaption></figcaption></figure>

Instruction  9: Moves rax into rbp

<figure><img src="../.gitbook/assets/image (307).png" alt=""><figcaption></figcaption></figure>

Instruction 14: This instruction assigns value of sample2 variable in RSI.

<figure><img src="../.gitbook/assets/image (308).png" alt=""><figcaption></figcaption></figure>

On stepi we'll see the change

<figure><img src="../.gitbook/assets/image (309).png" alt=""><figcaption></figcaption></figure>

Instruction 19: Changes sample variables 1 byte with that of al.&#x20;

<figure><img src="../.gitbook/assets/image (310).png" alt=""><figcaption></figcaption></figure>

Notice al has 88 right now and sample starts with 0xaa. This gets overwritten

<figure><img src="../.gitbook/assets/image (311).png" alt=""><figcaption></figcaption></figure>

Instruction 23: LEA would load 0x402000 intro RAX. Note that this is the memory address of sample variable.

<figure><img src="../.gitbook/assets/image (312).png" alt=""><figcaption></figcaption></figure>

Upon stepi or si, we see RAX being overwritten with the address of sample

<figure><img src="../.gitbook/assets/image (313).png" alt=""><figcaption></figcaption></figure>

Instruction 24: This instruction (lea rbx, \[rax]) essentially loads the value in RAX into RBX

<figure><img src="../.gitbook/assets/image (315).png" alt=""><figcaption></figcaption></figure>

Instruction 27: This instruction would exchange RAX and RBX. Notice how rax and rbx have been overwritten first by 64 bit absolute values

<figure><img src="../.gitbook/assets/image (316).png" alt=""><figcaption></figcaption></figure>

One more stepi

<figure><img src="../.gitbook/assets/image (317).png" alt=""><figcaption></figcaption></figure>

Finally, we exit the program using 0x3c (hex value for 60->syscall number for exit()) with rdi as 0 for error code.

<figure><img src="../.gitbook/assets/image (318).png" alt=""><figcaption></figcaption></figure>

