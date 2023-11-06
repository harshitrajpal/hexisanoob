# CTF: Memory Dereferencing

I encountered a CTF where I was provided with a binary which is taking a number as input. The expected outcome of the binary is that whenever a right number is entered, a flag is given out.

The problem is at any number input, a segmentation fault is given!

<figure><img src="../../.gitbook/assets/image (214).png" alt=""><figcaption></figcaption></figure>

Now, segmentation fault happens when program is trying to access a memory address that it's not allowed to. [https://www.geeksforgeeks.org/segmentation-fault-c-cpp/](https://www.geeksforgeeks.org/segmentation-fault-c-cpp/)

So, a good assumption is whatever I am entering, the program is considering it as an address. I decompiled it in Ghidra and saw what was happening below

<figure><img src="../../.gitbook/assets/image (216).png" alt=""><figcaption></figcaption></figure>

I noticed some key things here.&#x20;

1. A variable plVar2 was being assigned with the casted value of get\_number()
2. Another variable lVar1 was dereferencing the pointer `plVar2`

Read up about type casting: [https://ecomputernotes.com/what-is-c/function-a-pointer/type-casting-of-pointers](https://ecomputernotes.com/what-is-c/function-a-pointer/type-casting-of-pointers)

3. Then finally an if condition compares the second variable to a value 0xd000dfaceee

So this is what is happening in a nutshell:

1. A number is input and when it is typecasted, it is holding in a memory address. For example if 123 is input, plVar2 is now holding 0x7B
2. lVar1 is dereferencing that value to and storing whatever is there at that address in it.
3. lVar1 is compared to "0xd000dfaceee" and a flag is printed

**Actions**: We need to input a number which when converted to a memory address holds the value of 0xd000dfaceee

Let's check disassembly and see if we can locate 0xd000dfaceee.

<figure><img src="../../.gitbook/assets/image (217).png" alt=""><figcaption></figcaption></figure>



We see that at memory address 0x004008cee a mov instruction is given where 0xd000dfaceee is being moved to RAX.

The hex for this instruction (MOV RAX,0xd000dfaceee) is:

004008ce           48 b8 ee ce fa 0d 00 0d 00 00



Since the memory layout is in little endian, 004008ce + 2 bytes is where "0x0000d000dfaceee" is stored which is equivalent to "0xd000dfaceee "

0x004008ce + 2 bytes in memory would be: 0x004008d0



I converted this to deecimal using python

<figure><img src="../../.gitbook/assets/image (219).png" alt=""><figcaption></figcaption></figure>

When I input this number in the binary, I found the flag!!!

<figure><img src="../../.gitbook/assets/image (220).png" alt=""><figcaption></figcaption></figure>

