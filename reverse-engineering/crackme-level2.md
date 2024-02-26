# Crackme: level2

Author: nimacpp

Here is the decompiled binary:

<figure><img src="../.gitbook/assets/image (403).png" alt=""><figcaption></figcaption></figure>

So,  the program is comparing a string 01234567891 with some other string. Which is never equal. So what to do??!!

A hint is: look at it as a cracker

Uhmm, okay.



Here, strcmp is called then test eax,eax is done. I simply break at test, make  rax=0, test would return 0 and strcmp would be true. EZZZ

<figure><img src="../.gitbook/assets/image (404).png" alt=""><figcaption></figcaption></figure>



