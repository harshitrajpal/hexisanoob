# Virtual Memory and Paging

Ring 0 - OS level program

Ring 3 - User level program

[https://stackoverflow.com/questions/18717016/what-are-ring-0-and-ring-3-in-the-context-of-operating-systems](https://stackoverflow.com/questions/18717016/what-are-ring-0-and-ring-3-in-the-context-of-operating-systems)

\-----------------------------------------------------------------------------------------------------

Often while running different binaries, we see that they have some function addresses overlapping. So when we run both these binaries at the same time, it technically should cause problems, since two binaries can't have data at the same address!

So then why are these addresses same?

When we run "info proc mappings" on the binary, we can see where the binary is in memory and if it is using any libraries, addresses to those too. Here is gdb on a binary printing out stack information.

Page Mappings:

<figure><img src="../../.gitbook/assets/image (5) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

In Linux, one can see these mappings for any programs using /proc/self/maps

<figure><img src="../../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

What is a page? We'll see that below.



## Virtual Memory

The reason why different programs can use the same addresses is that these address don't correspond to the location of the program in physical memory. Operating system creates virtual overlays over physical memory and maps 0x40000 address in one program to let's say 0xdeadbeef in physical memory and 0x40000 address in other program to let's say 0xf00dface in physical memory.

This is mapped under the hood by OS using "pages." But when we debug a program, we will only see the virtual memory addresses.

Page of memory varies in size. Typically it is 4096 bytes. Recently people have started to use larger pages because it is slightly more efficient while dealing with large amount of data.

User level programs aren't allowed to change these mappings otherwise one could write in a space allocated to a different program.

Pages are also aligned. If you see in the screenshot above, all pages are starting at 0x...000 and ending at 0x...000 as well.

Pages have permissions too. In screenshot above, we can see R/W/X permissions on program. If a page permission is altered in lieu to read/write/execute on a particular address, CPU matches it with the permissions on physical memory, and throws an exception.

**Segmentation Fault:** CPU enforcing this protection where you're trying to access a page in a way you're not allowed to.

Note: Again in the screenshots you'll see permissions are either writable or executable but not both. This is to prevent unauthorized use of the memory so that an attacker can't write their own code in memory and then have it executed. This is called as unauthorized access to stack/heap protection.

