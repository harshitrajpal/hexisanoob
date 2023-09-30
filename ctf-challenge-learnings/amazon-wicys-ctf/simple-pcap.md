---
description: 'Goal: Can you spot the activity'
---

# Simple PCAP

I opened this PCAP file in Wireshark.&#x20;

<figure><img src="../../.gitbook/assets/image (196).png" alt=""><figcaption></figcaption></figure>

There was some activity going on in the TCP stream. I saw a PNG file being travelled.

<figure><img src="../../.gitbook/assets/image (197).png" alt=""><figcaption></figcaption></figure>

I extracted this using the following option:

<figure><img src="../../.gitbook/assets/image (198).png" alt=""><figcaption></figcaption></figure>

I saved the multipart/form-data 33 byte packet but it didn't work.

Then I observed the file being travelled in parts. I picked the specific PNG part in the wireshark explorer

<figure><img src="../../.gitbook/assets/image (199).png" alt=""><figcaption></figcaption></figure>

And then exported the bytes this like this and saved it as PNG file:

<figure><img src="../../.gitbook/assets/image (200).png" alt=""><figcaption></figcaption></figure>

The PNG had a flag!

<figure><img src="../../.gitbook/assets/image (201).png" alt=""><figcaption></figcaption></figure>



