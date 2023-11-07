# Intro to Buffer Overflow

Some inherently insecure functions exist in various programming languages that might help an attacker conduct buffer overflow and read/write/execute to or from a memory location that originally didn't allow a user to interact.

Example: gets() in C

<figure><img src="../../.gitbook/assets/image (246).png" alt=""><figcaption></figcaption></figure>

gets() function will just keep on reading data from a user even though the buffer we're trying to take input in is of restricted size like 32 bytes.



Exploitation is in CTF challenge learnings section
