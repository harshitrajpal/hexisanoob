# Task 1 - Tamper strcmp logic

Here, in a demo application I am making the application print a string on a wrong argv\[1] input.

The application prints "Welcome to the SLAE 64-bit course..."  string on the correct password entered. If not, it prints "it's time to review ..."

Here is the C code:

<figure><img src="../../.gitbook/assets/image (297).png" alt=""><figcaption></figcaption></figure>

Here, I have to tamper  with strcmp logic to make it true. This can be done by breaking at strcmp and changing RAX to 0 and continuing but processors sometimes don't allow that. A simple solution is just to change variable p to argv\[1] input and voila.

<figure><img src="../../.gitbook/assets/image (298).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (299).png" alt=""><figcaption></figcaption></figure>

