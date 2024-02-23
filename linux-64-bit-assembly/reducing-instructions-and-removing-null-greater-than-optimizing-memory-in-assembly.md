# Reducing instructions and Removing NULL-> Optimizing memory in Assembly

Often, like in the program we wrote on the previous page, we will observe that extra memory and null bytes  are being used. For example, in the hello program if we use objdump we  can see the bytes used for each instruction. Here, see how the first instruction is using 5 bytes for a simple "1" assigned to RAX.

<figure><img src="../.gitbook/assets/image (10).png" alt=""><figcaption></figcaption></figure>

In line 1,  we can see even though assembler has optimized my rax assignment of 1 to EAX (To save extra 32 bits), still 3 null bytes are there. We can further reduce this by using lower level register of RAX which is AL (name of 0 to 7 bits of RAX).

&#x20;

<figure><img src="../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Changing RAX to AL still produces the same output but as we can see in objdump, memory used is reduced from 5 to 2 bytes.

<figure><img src="../.gitbook/assets/image (2) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Similarly for each rdi,rsi and rdx we can do this and save bytes.

Is there a caveat here though?



## Shellcoding

When shellcoding, we can't have stray bytes in important registers.  So often times we'll see that shellcoders are using XOR to nullify the register and then assign it. This is genius since XOR between the same register only takes up 3 bytes and provides some assurance that the shellcode won't hit up garbage byte that may terminate the execution.

For example, here we can first XOR rax with rax and then assign al to 1. It consumes 5 bytes again but gives assurance. On a larger perspective we can still save bytes by reducing instructions to lower-level bytes of registers while nullifying the important ones of shellcode.

<figure><img src="../.gitbook/assets/image (3) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

In Objdump it looks like this:

<figure><img src="../.gitbook/assets/image (4) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

We can verify this in GDB and checking if RAX becomes 00000..

