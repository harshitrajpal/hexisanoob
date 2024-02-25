# GDB TUI Mode

Text User Interface makes it easier to look at code, set points, jump etc. It basically opens up multiple terminals on the same binary.

<figure><img src="../../.gitbook/assets/image (5) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

To display the assembly on the source terminal, we can use "layout asm"

<figure><img src="../../.gitbook/assets/image (2) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

While doing this, we can also display a list of registers using "layout regs"

<figure><img src="../../.gitbook/assets/image (3) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

All commands are shown under "help layout"

<figure><img src="../../.gitbook/assets/image (302).png" alt=""><figcaption></figcaption></figure>

Then we can proceed and step through the program normally and see registers change in real time. This makes our life easier!

<figure><img src="../../.gitbook/assets/image (303).png" alt=""><figcaption></figcaption></figure>

