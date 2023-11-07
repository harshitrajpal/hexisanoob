# Shoddy\_CMP

The binary presented is taking in a text input. Nothing fishy at first.

<figure><img src="../.gitbook/assets/image (229).png" alt=""><figcaption></figcaption></figure>

By decompiling the code in ghidra, I saw that it is taking in input in a 32 character array using gets() which doesn't know any bounds to input. This is the overflow injection point.

<figure><img src="../.gitbook/assets/image (230).png" alt=""><figcaption></figcaption></figure>

But even after overflow, there is no where to go.

This usually means that there is a different function running where IP could jump to to aid us in getting the flag. Upon little inspecting I found get\_time() function.

<figure><img src="../.gitbook/assets/image (231).png" alt=""><figcaption></figcaption></figure>



Let's do a quick due dilligence first. checksec tells us that there is no stack canary!

<figure><img src="../.gitbook/assets/image (232).png" alt=""><figcaption></figcaption></figure>

get\_time is at this memory address:\
\


<figure><img src="../.gitbook/assets/image (233).png" alt=""><figcaption></figcaption></figure>

Next, I figured out the rip offset which was 40 (by hitting and trying bytes one by one after 32)

I had some fun and attached GDB using pwntools:\


<figure><img src="../.gitbook/assets/image (234).png" alt=""><figcaption></figcaption></figure>

I'm using GDB enhanced features (github). Now when it opens up, I create various different length payloads which I automated using the script and passed as arguments when GDB opened up but for simplicity here is how I found the offset:

<figure><img src="../.gitbook/assets/image (235).png" alt=""><figcaption></figcaption></figure>

I input a pattern of 40 bytes with 6 B's and now I fully control the rip. Upon entering any more bytes, the segmentation fault was hitting end of the main function at return (makes sense, obv).

<figure><img src="../.gitbook/assets/image (237).png" alt=""><figcaption></figcaption></figure>

Anyway, upon entering 40 bytes + 6 B's here is what I get. A fully controlled rip

<figure><img src="../.gitbook/assets/image (236).png" alt=""><figcaption></figcaption></figure>

Now, let's try and make RIP jump to get\_time. I set a breakpoint at get\_time to see if RIP jumps and hits that. I'm using p64 which will convert the expected address in 64bit binary format.

<figure><img src="../.gitbook/assets/image (239).png" alt=""><figcaption></figcaption></figure>

And boom! Just like that, we are at get\_time now.

<figure><img src="../.gitbook/assets/image (242).png" alt=""><figcaption></figcaption></figure>

Cool. So, RIP is under our control. What else can we do? Let's inspect get\_time function in ghidra for ease. We see that there are various operations happening here.

<figure><img src="../.gitbook/assets/image (241).png" alt=""><figcaption></figcaption></figure>

Interesting. We see here at CMP instruction that RBP is being compared with string 0x1337. If it is 0x1337, it is jumped to LAB\_004006c0 and system() executes /bin/sh.

But when get\_time starts, if we see carefully, 0xdead is assigned to EBX using MOV. So, technically, /bin/sh will never be assigned to EBX.

So, what can we do?

If we somehow make RIP point to memory address 004006bb, the JNZ condition will be bypassed, EBX will be assigned with "/bin/sh" and then RIP will jump to next instruction which is LAB\_004006c0 where current RBX (or EBX) is assigned to RDI and system() is called with RDI as argument (which makes it system("/bin/sh") instead of system("/bin/date"))!!

So, let's do this.

<figure><img src="../.gitbook/assets/image (243).png" alt=""><figcaption></figcaption></figure>

I ran the script, hit continue on gef, and what do you know!! GDB is detached since a shell is executed and we received a local shell (on p.interactive() mode in pwn) where I can read the sample flag I created.

To do this on remote machine, I made a few alterations in the code like slapping in \r\n and using recvline() to wait for the entire response. Logic remains the same. I'm using remote() in pwntools to make connection

<figure><img src="../.gitbook/assets/image (244).png" alt=""><figcaption></figcaption></figure>

When I run this, I receive an interactive shell and can successfully read the flag!

<figure><img src="../.gitbook/assets/image (245).png" alt=""><figcaption></figcaption></figure>



