# Buffer Overflow Defenses

1. ASLR randomization: To prevent an attacker from predicting values of ROP gadgets and run overflow attacks, we can turn on ASLR randomization. This basically changes the address of operations in a program every time it is run. For example, 0xAAAAAAAA is the address where JMP ESP lies. 0xBBBBBBBB is where /bin/sh lies. 0xCCCCCCCC is where a system(char \*string) lies, attacker can simply take 0xAAAAAAAA to jmp esp to 0xBBBBBBBB which takes in string /bin/sh as input, then hits return call to execute it in 0xCCCCCCC. With ASLR turned ON, this issue goes away since attacker can no longer guess the memory of JMP ESP next time program will be run. Enabled by dedfault in modern compilers!!!
2. No Execution bit turned on. Makes stack non executable. Can be bypassed using ROP Gadgets technique. ROP Gadget are small operations that end in return calls. Read about ROP (return oriented programming): [https://codearcana.com/posts/2013/05/28/introduction-to-return-oriented-programming-rop.html](https://codearcana.com/posts/2013/05/28/introduction-to-return-oriented-programming-rop.html)\
   A tool called ROPGadgets on github can be used to find these ROP Gadgets in a binary.
3.  Random Canary = here, a random value is inserted by a compiler at the end of every stack frame in a program. It verifies canary before returning function. If an attacker tries to overflow it, when program returns, the function detects that the random value is now changed and quits execution. Canary is added by the compiler as a prevention mechanism against BufOF in the code. **To Still make it work, attacker needs to learn this random string.** Enabled by default in modern compilers!!\
    Drawbacks: Can not prevent integer overflow, heap overflow.\


    <figure><img src="../.gitbook/assets/image (1) (1) (2).png" alt=""><figcaption></figcaption></figure>

### Attack against Canary: Canary Extraction

char a\[10];

Let's assume the canary is "CANARY"

Now attacker inputs a string "A"\*10 + A ->program crash

"A"\*10 + B -> program crash

"A"\*10 + C -> Doesn't crash

<figure><img src="../.gitbook/assets/image (3) (1) (2).png" alt=""><figcaption></figcaption></figure>

This can continue until return address is hit to extract full canary



### Protections (contd...)

4. Stack Guard -> For buffer overflow
5. PointGuard -> For heap overflow
6.  ASAN -> Detects use after free (like double free) attacks.\


    <figure><img src="../.gitbook/assets/image (2) (2).png" alt=""><figcaption></figcaption></figure>
