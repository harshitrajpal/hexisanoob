# Endianness

Let's say  a hex string: 0x1122334455667788

In  Little Endian: 0x8877665544332211

In Big-Endian: 0x1122334455667788



<figure><img src="../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

x86 and x86-64 both use Little-Endian formats

Example

<figure><img src="../.gitbook/assets/image (3) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../.gitbook/assets/image (5) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

We can now jump to next instruction and examine rax

<figure><img src="../.gitbook/assets/image (4) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

This concludes that the CPU I am using is little endian in nature.
