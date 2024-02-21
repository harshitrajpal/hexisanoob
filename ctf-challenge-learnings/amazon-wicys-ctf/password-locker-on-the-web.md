# Password Locker on the Web

Upon entering some random data, an encrypted string was being printed out.

<figure><img src="../../.gitbook/assets/image (2) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Since, upon changing the string length, output string changed so I knew it wasn't a hash. It was definitely encryption.&#x20;

I first used burpsuite to break the 20 character limit and input 80 "A"s

<figure><img src="../../.gitbook/assets/image (3) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Turned out that the string became longer and 20 character limit was broken. Same could be obtained by changing length using developer tools.

<figure><img src="../../.gitbook/assets/image (4) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

I analyzed the string to find which kind of encryption was happening using [https://www.dcode.fr/cipher-identifier](https://www.dcode.fr/cipher-identifier)

<figure><img src="../../.gitbook/assets/image (5) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Upon negating the options which can't be a possibility, I shortlisted it to be XOR cipher. The key was a bunch of A's that we input!

<figure><img src="../../.gitbook/assets/image (6) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

