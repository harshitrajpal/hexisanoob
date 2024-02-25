# Introduction and Common Rules

Shellcode-> Machine code with a specific purpose like spawning shell/creating new accounts etc.

Shellcode-> Can be executed by the CPU directly without further assembling/linking or separate compiling

Shellcode-> Can be delivered as part of an exploit or by adding it to an executable file.

Shellcode-> Smaller the better!

Shellcode-> Bad characters should not be there like 0x00

Shellcode-> If shellcode is in an EXE, shellcode can run as a spearate thread or may even replace the functionality of the executable itself.

Shellcode resources-> exploit-db, shell-storm, projectshellcode



### Shellcode Testing

For the safe testing of a shellcode we create a safe environment like a C file and test it there. Here is a skeleton file:

```
#include<stdio.h>
#include<string.h>

unsigned char code[] = \
"<SHELLCODE>";


main()
{

	printf("Shellcode Length:  %d\n", (int)strlen(code));

	int (*ret)() = (int(*)())code;

	ret();

}

	
```

Whatever shellcode we want to test goes in here. Here, I am testing this shellcoded which runs /bin/sh

"\x31\xc0\x48\xbb\xd1\x9d\x96\x91\xd0\x8c\x97\xff\x48\xf7\xdb\x53\x54\x5f\x99\x52\x57\x54\x5e\xb0\x3b\x0f\x05"

Compilation command: gcc -fno-stack-protector -z execstack



<figure><img src="../../.gitbook/assets/image (11).png" alt=""><figcaption></figcaption></figure>

Upon running, it should give a /bin/sh



