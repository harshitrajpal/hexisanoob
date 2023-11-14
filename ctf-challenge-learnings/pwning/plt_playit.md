---
description: Deals with GOT overwrite by utlizing Write-What-where primitive
---

# PLT\_PlayIT

<figure><img src="../../.gitbook/assets/image (247).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (248).png" alt=""><figcaption></figcaption></figure>

In this binary, we can see in line 18, local\_28 bariable (which is only 8 bytes) is allowed to write 24 bytes in the memroy. What this will essentially do is overwrite local\_20 and local\_18 variables too which are the subsequent 8 bytes each variable in memory (making total 24 bytes).

But we see that local\_18 is a pointer. So we can overwrite it's value with an address.

Now in line 20, we see the value of local\_18 is assigned as local\_28. So at the address pointed by local\_18, local\_28 will be written to. And look at line 21, at the address pointed by local\_18\[1] (as in local\_18 + 8 bytes) local\_20 is being written to.

This allows us to utilize the "Write What Where" exploit primitive. We can write essentially 2 values in memory.

local\_28 -> Written in addr (\*local\_18)

local\_20 -> Written in addr + 8 (local\_18\[1])



Input would be: local\_28 (8 bytes) + local\_20 (8 bytes) + local\_18 (8 bytes)



So, essentially we have to craft a payload which would:

1. Write a malicious shellcode (or shell giving string) to a location
2. Have it called using system()
3. Overwrite GOT of a function by system. SHould be a function which comes after the initialization of local\_28, local\_20 and local\_18

Let's write "/bin/sh" somewhere in memory whose next 8 bytes and should be overwritten by system's PLT stub function.

This makes it simple. Only puts comes after initialization and is operating on local\_28. So, local\_28 becomes "/bin/sh". puts() is to be replcaced with system()'s PLT stub address

Upon exploring Ghidra I see one such prominent location at address 00601010:

<figure><img src="../../.gitbook/assets/image (249).png" alt=""><figcaption></figcaption></figure>

So, the payload becomes: "/bin/sh" + PLT STUB address of System() + 0x00601010

This would mean that 00601010 would be written with "/bin/sh"

00601010 + 8 (which is puts()) would be overwritten by PLT stub of system()

Essentially when puts(local\_28) will be called, system(local\_28) will execute.



System@plt is at:

<figure><img src="../../.gitbook/assets/image (250).png" alt=""><figcaption></figcaption></figure>

The same could be done using pwntools:

<figure><img src="../../.gitbook/assets/image (251).png" alt=""><figcaption></figcaption></figure>



So, exploit becomes: "/bin/sh" + 0x00000000004005d0 + 0x0000000000601010

I am representing the addresses in 64 bit value (8 byte addresses)

Let's whip up pwntools in python3 and let me guide you through my exploit dev

1. Made the payload but only interacting with binary intially to see how it reacts first before sending in the payload. Adding a \x00 terminated the string (/bin/sh) and is exactly 8 bytes long too.

<figure><img src="../../.gitbook/assets/image (253).png" alt=""><figcaption></figcaption></figure>

2. It is taking input after it says "to save: ". Let's incorporate that in code

<figure><img src="../../.gitbook/assets/image (254).png" alt=""><figcaption></figcaption></figure>

3. After it says that, I have to give an input. I use sendline to provide payload as an input.

<figure><img src="../../.gitbook/assets/image (255).png" alt=""><figcaption></figcaption></figure>

4. But where is the shell? The process breaks the execution after input so we can use interactive() to interact with the process after our input.

<figure><img src="../../.gitbook/assets/image (256).png" alt=""><figcaption></figcaption></figure>

But let me first set a SUID bit on this binary and run the exploit as a low priv user, so that when exploit works, I get a root shell. I'll also add a flag.txt file that can only be read by the root user.

<figure><img src="../../.gitbook/assets/image (257).png" alt=""><figcaption></figcaption></figure>

Let me also set SUID on local.py

5. Now it looks like this:

<figure><img src="../../.gitbook/assets/image (258).png" alt=""><figcaption></figcaption></figure>

6. Cool, let's give it a whirl.

<figure><img src="../../.gitbook/assets/image (259).png" alt=""><figcaption></figcaption></figure>

Let's run this on our remote server with a few alterations



<figure><img src="../../.gitbook/assets/image (261).png" alt=""><figcaption></figcaption></figure>



<figure><img src="../../.gitbook/assets/image (260).png" alt=""><figcaption></figcaption></figure>

There we go. That's how I pwned playit using write-what-where primitive to overwrite GOT for puts and replace by system() to get the flag eventually!

