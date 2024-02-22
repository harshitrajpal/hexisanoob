# Bitwise Logical Operations

Previous article talked about arithmetic operations. Here we will use bitwise logical operations like AND, OR, NOT, XOR etc.

Code used:

```


global _start			

section .text
_start:

	; NOT Operation 

	mov rax, qword [var2]
	not rax

	mov rbx, qword [var1]
	not rbx

	; AND Operation 

	mov rax, qword [var2]
	mov rbx, qword [var1]
	and rbx, rax

	mov rbx, qword [var1]
	and rbx, qword [var1]


	; OR Operation 

	mov rax, qword [var2]
	mov rbx, qword [var1]
	or rbx, rax

	mov rbx, qword [var1]
	or rbx, qword [var1]

	; XOR Operation

	mov rax, 0x0101010101010101
	mov rbx, 0x1010101010101010
	xor rax, rbx

	xor rax, rax

	xor qword [var1], rax


	; exit the program gracefully  

	mov rax, 0x3c
	mov rdi, 0		
	syscall

section .data

	var1 dq 0x1111111111111111
	var2 dq 0x0


```

As usual, nasm -f elf64 name.asm -o name.o

ld name.o  -o name

gdb -q ./name -tui



Notice there are two variables here var1 and var2

First operation performs NOT on rax and rbx by loading these values.

<figure><img src="../../.gitbook/assets/image (333).png" alt=""><figcaption></figcaption></figure>

As we can see, the value in var2 (0x0) is loaded in rax. Now, we not this and see that rax becomes 0xfffffffff.. since this is the logical NOT of 0x0000000...

<figure><img src="../../.gitbook/assets/image (334).png" alt=""><figcaption></figcaption></figure>

Then, we are loading 0x11111... in rbx and NOT it. It becomes 0xeeeeee...

<figure><img src="../../.gitbook/assets/image (335).png" alt=""><figcaption></figcaption></figure>

In the next two instructions we are setting rax as 0x000000... and rbx as 0x1111111..

<figure><img src="../../.gitbook/assets/image (336).png" alt=""><figcaption></figcaption></figure>

Next instruction is logical AND. As we can guess, 0x00000.. and 0x11111.. would make RBX 0x0000000

<figure><img src="../../.gitbook/assets/image (337).png" alt=""><figcaption></figcaption></figure>

Next, we are storing 0x11111.. in rbx and ANDing it with var1's value

<figure><img src="../../.gitbook/assets/image (338).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (339).png" alt=""><figcaption></figcaption></figure>

Also note ZF is now removed.

Next, we are storing 0x1111111... in rbx. RAX has 0x00000... and we are ORing them

<figure><img src="../../.gitbook/assets/image (340).png" alt=""><figcaption></figcaption></figure>

Since 1 or 0 is 1, rbx holds 0x1111..

Please note, OR RBX,RAX would OR the two registers and store the value in first register (RBX)

Next, we will set RAX to 0x0101010101... and RBX to 0x1010101010101010 and then XOR it. XOR rax,rbx would store result in rax. If two bits are different, XOR makes it 1.

<figure><img src="../../.gitbook/assets/image (341).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (342).png" alt=""><figcaption></figcaption></figure>

Then we XOR rax with rax to make it 0 (same bits XOR = 0)

<figure><img src="../../.gitbook/assets/image (343).png" alt=""><figcaption></figcaption></figure>

Finally, XORing var1 with rax (0x1111.. with 0x0000...) and storing result in var1

<figure><img src="../../.gitbook/assets/image (344).png" alt=""><figcaption></figcaption></figure>



