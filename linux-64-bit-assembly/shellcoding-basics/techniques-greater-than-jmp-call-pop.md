# Techniques-> JMP,CALL,POP

In this shellcode, we talk about a simple hello world program.

As opposed to just using simple assembly, we'll keep the shellcoding rules in our mind while we try to print hello world on the screen.

1. Remove  all 0x00s
2. Make sure there are no hardcoded addresses of "Hello World". So we have to dynamically try to figure out the address of "hello World"

Remember this hello world code from 2nd article?

```
global start

section .text

        _start:
        mov rax,1 ;syscall for write.
        mov rdi,1 ;int fd, which is 1 for stdout since we are not storign the  message anywhere rather printing it out
        mov rsi,hello_world ; The  buffer to be printed
        mov rdx,length ; The length of   hello world buffer we calculated in data section
        syscall ; Executes the syscall 1 with arguments we gave. At this point rax  will store the return value from the executed syscall
        mov rax, 60 ; syscall number for exit
        mov rdi, 11 ; Custom error code. Could be anything. 0 in most cases.
        syscall

section .data
        hello_world: db "Hello world! This is Harshit"
        length: equ $-hello_world
```

Here, we are hardcoding the address of hello\_world in rsi. In a shellcode we can not do that.

So, a JMP->CALL->POP technique would be used.

```
JMP call_shellcode
call_shellcode:
        call shellcode:
                HelloWorld: db "Hello World",0xa
shellcode:
        pop rsi
        ....other code.....
```

Why this?

Here is how it works: JMP would jump to the specific location where string is defined. That area has a "CALL" instruction which is used to call the shellcode section. This section has a POP instruction.

Remember when you use CALL, the address of that location gets stored on the stack. When we reach POP, we move that address from the stack back into RSI and then we can successfully print it out.

```
;structure would be:
;JMP->CALL->POP

global _start

section .text
_start:

        jmp call_shellcode

shellcode:
        pop rsi
        mov rax,1
        mov rdi,1
        mov rdx,12 ; length of hello world message
        syscall

        ;exits the program
        mov al,60
        xor rdi,rdi
        syscall



call_shellcode:
        call shellcode
        hello_world: db 'Hello World',0xa
```

<figure><img src="../../.gitbook/assets/image (4) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Upon inspecting the objdump, we can see there are a few 0s now that need to be removed.

<figure><img src="../../.gitbook/assets/image (7) (1) (1).png" alt=""><figcaption></figcaption></figure>

Here is the updated code



```
;structure would be:
;JMP->CALL->POP

global _start

section .text
_start:

        jmp call_shellcode

shellcode:
        pop rsi
        mov al,1
        mov rdi,rax ; Way to make rdi 1
        mov rdx,rdi ; length of hello world message
        add rdx,11 ; adding rather than assigning
        syscall

        ;exits the program
        xor rax,rax
        add rax,60 ; another way to avoid 0s
        xor rdi,rdi
        syscall
        
call_shellcode:
        call shellcode
        hello_world: db 'Hello World',0xa
```



<figure><img src="../../.gitbook/assets/image (8) (1).png" alt=""><figcaption></figcaption></figure>

Let's add this shellcode  in our C code and test

<figure><img src="../../.gitbook/assets/image (9) (1).png" alt=""><figcaption></figcaption></figure>

```
#include <stdio.h>
#include <string.h>
#include <sys/mman.h>

unsigned char code[] = \
"\xeb\x1b\x5e\xb0\x01\x48\x89\xc7\x48\x89\xfa\x48\x83\xc2\x0b\x0f\x05\x48\x31\xc0\x48\x83\xc0\x3c\x48\x31\xff\x0f\x05\xe8\xe0\xff\xff\xff\x48\x65\x6c\x6c\x6f\x20\x57\x6f\x72\x6c\x64\x0a";
int main() {

    size_t code_length = strlen(code);
    printf("Shellcode Length:  %zu\n", code_length);

    // Allocate a memory page that is both writable and executable
    void *mem = mmap(NULL, code_length, PROT_WRITE | PROT_EXEC, MAP_ANON | MAP_PRIVATE, -1, 0);
    if (mem == MAP_FAILED) {
        perror("mmap");
        return -1;
    }

    // Copy the shellcode into the allocated memory
    memcpy(mem, code, code_length);

    // Cast the allocated memory to a function pointer and execute it
    int (*ret)() = (int(*)())mem;
    ret();

    // Free the allocated memory (optional here since the OS will clean up upon process exit)
    munmap(mem, code_length);

    return 0;
}
```

gcc -fno-stack-protector -z execstack skeleton.c -o skeleton

<figure><img src="../../.gitbook/assets/image (10) (1).png" alt=""><figcaption></figcaption></figure>

This is how we can create this small shellcode.

<figure><img src="../../.gitbook/assets/image (7).png" alt=""><figcaption></figcaption></figure>

In stack pointer, we can see it initialized by 0x000000..1 right now.

<figure><img src="../../.gitbook/assets/image (2) (1) (1).png" alt=""><figcaption></figcaption></figure>

Notice how rsp changes when call shellcode is performed. It stores the memory address of the hello\_world array in the call\_shellcode section on stack

<figure><img src="../../.gitbook/assets/image (3) (1) (1).png" alt=""><figcaption></figcaption></figure>

Further notice what happens to rsi when pop is called. The memory address of hello\_world is stored in RSI now.

<figure><img src="../../.gitbook/assets/image (4) (1) (1).png" alt=""><figcaption></figcaption></figure>

And then it executes and exits.

<figure><img src="../../.gitbook/assets/image (5) (1) (1).png" alt=""><figcaption></figcaption></figure>



