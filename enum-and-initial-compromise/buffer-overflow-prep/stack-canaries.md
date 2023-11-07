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
*

