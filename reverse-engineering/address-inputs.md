# Address inputs

Binary: postage (ELF-64)

<figure><img src="../.gitbook/assets/image (391).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../.gitbook/assets/image (392).png" alt=""><figcaption></figcaption></figure>

it's the same get\_number function from  previous article.

Program is doing this so far:

1. Taking an input in var2. Assigning var1 = \*var2
2. if var1 != 0xd000ffaceee, exit or else print the flag.

Consider the pointers at play here. The input I give is being converted to a long integer using strtol so all the characters would be reducted.

Plus, the input I am giving in integer value, it's pointer referencec is being assigned to var1. So, var2 (user input) has  to be a memory address. Finally, it has to be an input address matching 0xd000dfaceee

SOOO



To sum it up: Input an address (in integer value) where the string "0xd000dfaceee" is.

Here it is:

It starts from address 004008d5 to 004008d0. Since we have to input this in reverse(stack,phew) address is: 0x4008d0

<figure><img src="../.gitbook/assets/image (393).png" alt=""><figcaption></figcaption></figure>

So, the input is

<figure><img src="../.gitbook/assets/image (394).png" alt=""><figcaption></figcaption></figure>



