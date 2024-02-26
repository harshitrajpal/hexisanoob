---
description: aka stack cookies
---

# Stack Canaries

The idea is that as long as we can put a secret value in between the stack variables and the return address, then we can check it before we return to the next instruction.

This is a mitigatioln of buffer overflow attacks

`PREAMBLE`&#x20;

`mov rax, qword fs:[0x28]`&#x20;

`mov qword [rbp-0x8], rax`&#x20;

`BEFORE RETURN`&#x20;

`mov rdx, qword [rbp-0x8]`&#x20;

`xor rdx, qword fs:[0x28]`&#x20;

`je 0x40079f`&#x20;

`call _stack_chk_fail`



fs is a special storage area which is difficult to access even if you have a vulnerability like information disclosure.

rbp-8 is right where local variables end



Just before returning, we'll pull that value back out from rbp-8 in stack, XOR it with original value in fs

If they are equal, then jump to the return address or else call "stack\_check\_fail" function.



* Nothing in the function is supposed to change the stack cookie. If it is changed this means that there is a buffer overflow. Stack check fail would print out "stack smashing detected" and will exit the program
* Stack canaries can still be avaded and attacks conducted by:
  * Stack Canary leaking
  * Stack canary bruteforcing - Guessing bytes of the canary one by one. Once the offset is known, we can start adding a single byte (00 to ff) and see which byte doesn't cause crash. THe byte that doesn't is the correct one. We can extend this for all bytes until the correct 64 bit number is known.
*   Stack canaries always start with a null byte. Because in C, strings are terminated by null bytes. so if an attacker tries to read canary, C would stop it right before it accesses the canary since null byte would be encountered. However, some C functions would still allow reading of this canary even with a null byte in place.\
    \
    \


    <figure><img src="../../.gitbook/assets/image (20).png" alt=""><figcaption></figcaption></figure>

Now when we step to next intruction where fs is being assigned to rax, we can see the canary

<figure><img src="../../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

See how it ends with 00? In little endian, when a program would read it, it would start backwards, so from 00 and the reading function would be terminated upon encountering the canary.





