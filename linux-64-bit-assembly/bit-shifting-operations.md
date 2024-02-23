# Bit-Shifting Operations

SHL/SAL -> Shift Left operation shifts the most significant bit to left one position.

<figure><img src="../../.gitbook/assets/image (345).png" alt=""><figcaption></figcaption></figure>

In this operation, Carry Flag is used to store the shifted bit. And as we can see the LSB is auto-populated with 0.

SHR -> Shift Right

<figure><img src="../../.gitbook/assets/image (346).png" alt=""><figcaption></figcaption></figure>

In this, LSB is shifted to carry flag and MSB is auto-populated with 0.



SAR-> Shift Arithmetic Right. This is very interesting. If the operand is positive, 0 is populated in the MSB but if it is negative, 1 is populated.

<figure><img src="../../.gitbook/assets/image (347).png" alt=""><figcaption></figcaption></figure>

ROR-> Rotate Right. Some bits are rotated from LSB to MSB by a particular value.



Here is a demo:

```


global _start			

section .text
_start:

	mov rax, 0x00000000ffffffff
	sal rax, 32
	sal rax, 1

	clc	
	mov rax, 0x00000000ffffffff
	shr rax, 1
	shr rax, 31

	clc
	mov rax, 0x00000000ffffffff
	sar rax, 1
	clc
	mov rax, 0xffffffffffffffff
	sar rax, 1

	clc
	mov rax, 0x0123456789abcdef
	ror rax, 8
	ror rax, 12
	ror rax, 44


	; exit the program gracefully  

	mov rax, 0x3c
	mov rdi, 0		
	syscall

section .data

	var1 dq 0x1111111111111111
	var2 dq 0x0


```

1. Shift left by 0x20 (32 bits decimal)

<figure><img src="../../.gitbook/assets/image (348).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (349).png" alt=""><figcaption></figcaption></figure>

2. SHL by 1 bit again.  See CF is now 1 as 1 bit from ff is taken which becomes fe.

<figure><img src="../../.gitbook/assets/image (350).png" alt=""><figcaption></figcaption></figure>

Continue and inspect this behavior.



