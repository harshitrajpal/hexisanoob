# Techniques-> Stack

We are considering the same hello world program.

In this technique we will treat "hello world" string as data, and push it on the stack in reverse order (as the stack goes from high to low in memory). Then we get this string as a reference from RSP.



1. Let's convert "hello world" string in hex using python

```
In [1]: import binascii

In [2]: message = "Hello World\n"

In [3]: message[::-1]
Out[3]: '\ndlroW olleH'

In [4]: len(message)
Out[4]: 12

In [5]: binascii.hexlify(message[::-1].encode())
Out[5]: b'0a646c726f57206f6c6c6548'
```

<figure><img src="../../.gitbook/assets/image (6) (1).png" alt=""><figcaption></figcaption></figure>

The hex equivalent in reverse for hello world is: 0a646c726f57206f6c6c6548

I tried pushing the entire thing at once and got this error:

<figure><img src="../../.gitbook/assets/image (7) (1).png" alt=""><figcaption></figcaption></figure>

Since message is 12 bytes, we need to break it down to 8 bytes and 4 bytes. Move first 4 bytes first onto the stack and then use a register to move the last 8 bytes.

<figure><img src="../../.gitbook/assets/image (8).png" alt=""><figcaption></figcaption></figure>

Good. This works. But there are a lot of 0s still

<figure><img src="../../.gitbook/assets/image (9).png" alt=""><figcaption></figcaption></figure>

So, I corrected it like so:

```
;structure would be:
;push string in reverse on stack
; 2nd technique of shellcode referencing without direct address usage

global _start

section .text
_start:

        xor rax,rax
        mov rdi,rax
        add rdi,1
        mov al,1
        push 0x0a646c72
        mov rbx, 0x6f57206f6c6c6548
        push rbx
        mov rsi, rsp
        xor rdx,rdx
        add rdx,12
        syscall


        xor rax,rax
        add rax,60
        xor rdi,rdi

        syscall
```

<figure><img src="../../.gitbook/assets/image (10).png" alt=""><figcaption></figcaption></figure>



Good, as usual adding the opcodes in skeleton to test it's functionality.

```
objdump -d ./2hello|grep '[0-9a-f]:'|grep -v 'file'|cut -f2 -d:|cut -f1-6 -d' '|tr -s ' '|tr '\t' ' '|sed 's/ $//g'|sed 's/ /\\x/g'|paste -d '' -s |sed 's/^/"/'|sed 's/$/"/g'
```







