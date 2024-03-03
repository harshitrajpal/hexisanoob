# Basic Shellcodes->Exit

Major thing to note is that when we use RAX and RDI with the SYSCALL, it leads to 0x00 in shellcode.

So, we need to change certain instructions to avoid 0x00

Also we need to make shellcode more compact. Like using AH/AL instead of RAX.



For example, here is a shellcode that exits the program. Simple.

```
global _start

section .text
_start:
        mov rax,0x3c
        mov rdi,0xb
        syscall
```

<figure><img src="../../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

As we see above that mov rax has been converted to mov eax to optimize memory. But if we look at objdump, we'll still note many 0's after mov eax,0x3c instruction (in hex b8 3c after we see 3 bytes of 0s). Similarly after moving to RDI, we see 3 bytes of 0x again.

<figure><img src="../../.gitbook/assets/image (2) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

So, we remove the 0's by replacing these instructions with equivalent instructions that do not produce 0s. We know both of these mov instructions take 2  bytes and the number we are inputting (60 and 11 or 0x3c and 0xb in hex are also 1 byte each). So ideally we can use lower 1 byte of RAX and RDI to do this.

Also, we don't need rdi to return 11. We can simply return 0 as well.

<figure><img src="../../.gitbook/assets/image (3) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

And if we look at objdump, 0s are gone!!

<figure><img src="../../.gitbook/assets/image (4) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

So, to create shellcode from this we have to extract these bytes in sequence. To automate this, here is a command line fu:

```
objdump -d ./PROGRAM|grep '[0-9a-f]:'|grep -v 'file'|cut -f2 -d:|cut -f1-6 -d' '|tr -s ' '|tr '\t' ' '|sed 's/ $//g'|sed 's/ /\\x/g'|paste -d '' -s |sed 's/^/"/'|sed 's/$/"/g'
```

<figure><img src="../../.gitbook/assets/image (5) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

I can insert this in my C skeleton code and run this to exit the program. BUT WAIT? DO you also see a segmentation fault? Bear with me and see the next article.

