---
description: 'Note: This technique is only for 64-bit variant'
---

# Techniques-> (64-bit only) RIP Relative Addressing

So, so far we know that a string can not be directly hardcooded in the assembly (eg: using mov rsi, hello\_world)

This  would hardcode the memory address of the string in shellcode and when we paste that in C, since addresses change, string won't be at the same location and we'd get a segmentation fault.



So far we have learnt 2 techniques to counter this. Here is a 64-bit variant for the same.



We want to include the string onto a register using RIP + offset. For example, If I  know my string is 13 instructions down below, I can simply so "lea rsi,\[rip+0x13]"

let's take that one step further. In assembly an instruction called "rel" exists which does relative RIP addresing. Here is an example:



```
global _start

section .text

_start:

        mov rax,1
        mov rdi,1
        lea rsi, [rel message]
        mov rdx,13
        syscall

        ;exiting
        mov rax,60
        mov rdi,0
        syscall

        message: db 'Hello World!',0xa

```

In objdump this looks something like this:

<figure><img src="../../.gitbook/assets/image (377).png" alt=""><figcaption></figcaption></figure>

Wonderful. But in shellcode we can't have 0s. Let's remove that.

<figure><img src="../../.gitbook/assets/image (378).png" alt=""><figcaption></figcaption></figure>

Well, we can see in objdump that majorly all 0s are now removed. But what to do with our hero line? The relative addressing. We can see that rip + 0x15 is nothing but rip+0x00000015

Trick: Since there are 0s within the relative addressing, we can move our message line above the lea line and change that from 0s to "f"s!!

<figure><img src="../../.gitbook/assets/image (379).png" alt=""><figcaption></figcaption></figure>

Voila! Since the negative offset is converted to F in hex, we removed the 0s.

```
global _start

section .text
message: db 'Hello World!',0xa
_start:

        xor rax,rax
        add rax,1
        mov rdi,rax
        lea rsi, [rel message]
        mov rdx,rdi
        add rdx,12
        syscall

        ;exiting
        xor rax,rax
        add rax,60
        xor rdi,rdi
        syscall
```

Here is the extracted shellcode:

```
objdump -d ./hello.o |grep '[0-9a-f]:'|grep -v 'file'|cut -f2 -d:|cut -f1-6 -d' '|tr -s ' '|tr '\t' ' '|sed 's/ $//g'|sed 's/ /\\x/g'|paste -d '' -s |sed 's/^/"/'|sed 's/$/"/g'
```

"\x48\x65\x6c\x6c\x6f\x20\x57\x6f\x72\x6c\x64\x21\x0a\x48\x31\xc0\x48\x83\xc0\x01\x48\x89\xc7\x48\x8d\x35\xe2\xff\xff\x48\x89\xfa\x48\x83\xc2\x0c\x0f\x05\x48\x31\xc0\x48\x83\xc0\x3c\x48\x31\xff\x0f\x05"





