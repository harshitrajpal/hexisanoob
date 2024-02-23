---
description: >-
  In C, we have strcmp and memcmp. Here, we have scas and cmps. Similarly we
  have load, store and move strings
---

# String Operations

Pre: cld->instruction to clear direction flag. If it is cleared, then when the addresses are copied from one location to the other, addresses are incremented. Else they are decremented.

* **DF=0 (Clear):** When the Direction Flag is clear, string operations process memory from lower addresses to higher addresses. This is known as "forward direction." Instructions like `MOVS`, `LODS`, `STOS`, `CMPS`, and `SCAS` will increment the index or pointer registers (such as `SI`, `DI`, `ESI`, or `EDI`) after each operation.
* **DF=1 (Set):** When the Direction Flag is set, string operations process memory from higher addresses to lower addresses. This is known as "backward direction." In this case, the index or pointer registers are decremented after each operation.

For example,

#### Initial Setup

* **Source String:** Located at memory address `0x100`, contents: `"ABCDEFGH"`
* **Destination String:** Located at memory address `0x110`, initial contents can be anything, let's say `"ZZZZZZZZ"`

We want to copy the source string to the destination string using the `MOVS` instruction.

#### Expected Behavior (with DF=0)

Let's first examine the expected behavior when the Direction Flag is cleared (DF=0), which means we're copying from lower to higher memory addresses.

1. **Before Operation:**\
   Source (0x100): `ABCDEFGH`\
   Destination (0x110): `ZZZZZZZZ`
2. **Operation:**\
   Execute `MOVS` instructions to copy each byte from source to destination.
3. **After Operation:**\
   Source (0x100): `ABCDEFGH` (unchanged)\
   Destination (0x110): `ABCDEFGH` (copied as expected)

#### Actual Behavior (with DF=1)

Now, let's see what happens if the Direction Flag is set (DF=1), and we accidentally copy from higher to lower addresses without realizing it.

1. **Before Operation (same as before):**\
   Source (0x100): `ABCDEFGH`\
   Destination (0x110): `ZZZZZZZZ`
2. **Operation (DF=1):**
   * The `MOVS` instruction starts copying from the end of the source string to the end of the destination string because the Direction Flag indicates backward copying.
   * This backward operation is not what we intended for a straightforward copy from one buffer to another.
3. **After Operation:**
   * The operation proceeds in reverse, starting with copying 'H' to the last position of the destination, then 'G' to the second last, and so on.
   * However, since we're expecting a forward copy, our code logic might not correctly handle this reverse operation, leading to confusion or bugs, especially in scenarios where buffer boundaries, overlapping regions, or specific processing logic are critically important.



In short, to ensure "abcdefgh" is copied to destination as "abcdefgh" and not something unintended (like example below) use cld before string operations.

## Comparing Strings

Input and comparisons of strings is done with "SCASX" and "CMPSX" where X is->B/W/D/Q respectively for 1,2,4,8 byte strings



SCASX-> Compares register al/ax/eax/rax (Depending if it is scasb,scasw,scasd,scasq) with memory referenced by RDI

CMPSX->   Compares memory referenced by RSI with RDI



So, SCAS-> memory to register comparison (source in RAX, to be compared with RDI)

CMPS-> memory-to-memory comparison (Source in RSI, to be compared with RDI)

```
; File for scasb, cmpsb etc.
global _start

section .text
_start:

        ; scasb/w/d/q
        ; Compare memory to register

        cld ; clear direction flag
        mov rax, 0x1234567890abcdef ; moves an absolute value in rax
        lea rdi, [var1] ; loads address of var1 in rdi
        scasq ; compares rax's absolute value with the value pointed to by rdi

        lea rdi, [var2]
        scasq

        ; cmpsb/w/d/q
        ; Compare memory to memory

        cld
        lea rsi, [var1]
        lea rdi, [var3]
        cmpsq

        ; exit the program

        mov rax, 0x3c
        mov rdi, 0
        syscall

section .data

        var1:   dq      0x1234567890abcdef
        var2:   dq      0xfedcba1234567890
        var3:   dq      0x1234567890abcdef
```



Notice in  the program that instruction 1 has moved an absolute value in rax. "lea" has loaded the address of variable 1

<figure><img src="../../.gitbook/assets/image (365).png" alt=""><figcaption></figcaption></figure>

Also notice how "scasq" has now become "scas" and is comparing rax with QWORD pointed to by RDI. This is an internal operating trick where "scasq" or "scasw" etc are interpreted and typecasted.

Upon stepping this instruction, we can see that the comparison has now been done and Zero Flags is set.

Zero Flag is set because the comparison was successful.

<figure><img src="../../.gitbook/assets/image (366).png" alt=""><figcaption></figcaption></figure>

Now, in the next instruction, the value to be compared is different. When SCAS is run, ZF is not set, indicating the comparison was false.

<figure><img src="../../.gitbook/assets/image (367).png" alt=""><figcaption></figcaption></figure>

Similarly, in CMPS, two memory addresses are compared. Comparison happens between RSI and RDI. Here, var1 and var3 are same to ZF is set.

<figure><img src="../../.gitbook/assets/image (368).png" alt=""><figcaption></figcaption></figure>

## Loading, Storing, Moving Strings

LODSX -> Stores from memory to register

MOVSX -> Moves strings from memory to memory

STOSX -> Stores from register to memory

Where X could bd b,w,d,q for 1,2,4,8 bytes respectively

Program:

```
global _start

section .text
_start:

        ; Movsb/w/d/q
        ; Memory to Memory
        cld
        lea rsi, [HelloWorld]
        lea rdi, [Copy]
        movsq

        ; for second demo
        cld
        lea rsi, [HelloWorld]
        xor rax, rax
        mov qword [Copy], rax
        ; till this point we are just zeroing out rax and Copy variable
        lea rdi, [Copy] ; copying destination memory address in rdi
        mov rcx, len ; we would like to loop through x times where x is the length of the string which is len (equated in .text)
        rep movsb ; loop though and copy byte after byte
        ; second demo ends

        ; stosb/w/d/q
        ; Register to Memory

        mov rax, 0x0123456789abcdef
        lea rdi, [var1]
        stosq

        ; lodsb/w/d/q
        ; Memory to Register

        xor rax, rax
        lea rsi, [var1]
        lodsq

        ; exit the program gracefully

        mov rax, 0x3c
        mov rdi, 0
        syscall


section .data

        HelloWorld:     db      "Hello World"
        len:    equ     $-HelloWorld

section .bss

        Copy:   resb    len
        var1:   resb    8
```



Upon running the first block (MOVS) in GDB we observe something.

1. The source is our RSI string which is the "Hello World" string that is to be copied.
2. The destination RDI is loaded with the address of "Copy"  variable is in our bss section. We have some unreserved memory location of the variable "Copy"

These registers are respectively loaded with memory addresses.&#x20;

<figure><img src="../../.gitbook/assets/image (369).png" alt=""><figcaption></figcaption></figure>

When we inspect the destination address initially, we see nothing there. After the movs instruction "Hello Wo" appears (8 bytes only since movsq is used)

<figure><img src="../../.gitbook/assets/image (370).png" alt=""><figcaption></figcaption></figure>

To copy the entire string, programmers often have to loop through the source string until every byte is copied. We can use rep for this and put a counter in rcx. Here is where cld comes in useful. We want to copy byte after byte in an incremental order!!

<figure><img src="../../.gitbook/assets/image (371).png" alt=""><figcaption></figcaption></figure>

Now, we loop len times and see the destination variable's value now. len is the length of "Hello World"

<figure><img src="../../.gitbook/assets/image (372).png" alt=""><figcaption></figcaption></figure>

Let's actually see what gets copied when we don't use cld by tweaking our program. I removed cld here and rather used std (to set the direction flag). Now cld should be equal to 1 which is decremental copying.  As we see, the program just keeps copying the first byte again and again.

<figure><img src="../../.gitbook/assets/image (375).png" alt=""><figcaption></figcaption></figure>

This makes the program decrement addresses as it proceeds making it only copy "H" which is the first byte of the message.

<figure><img src="../../.gitbook/assets/image (374).png" alt=""><figcaption></figcaption></figure>

This is happening because Direction Flag is forcing the decrement and the next byte after "H" is not reached at all.

Next, STOS is used to copy from register RAX to memory 0x402017

<figure><img src="../../.gitbook/assets/image (376).png" alt=""><figcaption></figcaption></figure>

And the opposite with LODS.

