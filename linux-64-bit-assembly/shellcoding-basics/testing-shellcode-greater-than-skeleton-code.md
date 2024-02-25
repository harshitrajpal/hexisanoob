# Testing shellcode->Skeleton Code

In previous articles, I mentioned testing shellcode in a skeleton C code. Here is an example of "exit" shellcode in the C skeleton.

```
#include<stdio.h>
#include<string.h>

unsigned char code[] = \
"\x48\x31\xc0\xb0\x3c\x48\x31\xff\x0f\x05";


int main()
{

        printf("Shellcode Length:  %d\n", (int)strlen(code));

        int (*ret)() = (int(*)())code;

        ret();

}
```

When I compile this and run, this happens:

<figure><img src="../../.gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>

This is always bound to happen using modern systems. I tried identifying the problem and here is a guess.  The shellcode lands in the .data section in the assembly after the assembler works. This .data segment is essentially non-executable.

<figure><img src="../../.gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>

So, to test the shellcode properly using a skeleton, we have to resort to keeping the code in a memory section that is executable. After som suggestions, using mmap() to allocate a page of memory which is executable makes sense. So the skeleton code is:



```
#include <stdio.h>
#include <string.h>
#include <sys/mman.h>

unsigned char code[] = \
"\x48\x31\xc0\xb0\x3c\x48\x31\xff\x0f\x05";

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

<figure><img src="../../.gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>

In GDB, we can inspect and confirm this functionality. AS we seee, the code exited normally

<figure><img src="../../.gitbook/assets/image (3).png" alt=""><figcaption></figcaption></figure>





