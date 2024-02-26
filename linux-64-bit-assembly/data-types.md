# Data Types

Here I am introducing different data types in assembly. Read  the comments to know more.

```
;note how I'm using different data types here and the brackets to initialize rax
;with not the address of a variable but the absolute value
global start

section .text

        _start:
        xor rax,rax; nullifying rax to make sure no garbage bytes exist
        mov al,1 ;syscall for write.
        mov rdi,1 ;int fd, which is 1 for stdout since we are not storign the  message anywhere rather printing it out
        mov rsi,hello_world ; The  buffer to be printed
        mov rdx,length ; The length of   hello world buffer we calculated in data section
        syscall ; Executes the syscall 1 with arguments we gave. At this point rax  will store the return value from the executed syscall

        mov rax, var4 ; Loads up the var4 address in rax
        mov rax, [var4] ; Loads up the value in var4 in rax

        mov rax, 60 ; syscall number for exit
        mov rdi, 11 ; Custom error code. Could be anything. 0 in most cases.
        syscall

section .data
        hello_world: db "Hello world! This is Harshit",0x0a
        length: equ $-hello_world

        var1: db 0x11,0x22 ; Assigns 2 bytes 0x11 and 0x22 to var1
        var2: dw 0x3344 ; Assigns a word (2 bytes) to var2
        var3: dd 0xaabbccdd ; Assigns a double word (4 bytes) to var3
        var4: dq  0xaabbccdd11223344 ; Assigns a quad word (8 bytes) to var4

        repeat_buffer: times 128 db 0xAA ; Assigns 128 bytes of 0xaa to repeat_buffer


section .bss
        buffer: resb 64 ; Reserves uninitialized 64 bytes
```

After assembling and linking the code, we can dissect this in GDB and check what is happening.

1. Here, we can see using "info  variables" our variables are initialized

<figure><img src="../.gitbook/assets/image (6) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

2. In GDB, we can use "x" to examine these. Use help x to display what commands can be used. So, x/x means examine in hex. Then add if you want to see bytes   or whatever in x/\<size>x\<keyword> REGISTER\_NAME format

<figure><img src="../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

3. We can also examine repeat\_buffer (that times initialized with 128 0xaa) and buffer that initializes 64 bytes unreserved (assembler by default assigns it with 0x00)

<figure><img src="../.gitbook/assets/image (4) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

4. We should run this and set a breakpoint on lines where I am assigning \[var4] and var4 to rax to see how registers are behaving when RIP reaches there

<figure><img src="../.gitbook/assets/image (5) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Upon hitting breakpoint 2, we can observe that the memory address of variable 4 (var4) is not yet loaded in the rax register. Using "nexti" I can make the RIP jump one point and make it execute the next intstruction.&#x20;

<figure><img src="../.gitbook/assets/image (6) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

When I do nexti, I hit breakpoint3. This is the breakpoint where memory address of var4 has been set to rax but "mov rax \[var4] is yet to be set. At that point, we can see registers and conclude var4's memory address has been set.&#x20;

<figure><img src="../.gitbook/assets/image (7) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Now, we can examine the memory address in RAX right now and confirm that it is infact the var4 (same string). With "nexti" I jump RIP to next instruction and at that moment, "mov rax \[var4]" has been executed and RAX assigned with the 8 byte string.

<figure><img src="../.gitbook/assets/image (8) (1) (1).png" alt=""><figcaption></figcaption></figure>

We can  use disas command to see where RIP is at the moment. (notice arrow on the left)

<figure><img src="../.gitbook/assets/image (9) (1) (1).png" alt=""><figcaption></figcaption></figure>

