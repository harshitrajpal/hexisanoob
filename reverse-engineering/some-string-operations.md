# Some string Operations

strops.bin

Upon  opening it in Ghidra we see this:

<figure><img src="../.gitbook/assets/image (3).png" alt=""><figcaption></figcaption></figure>

I copied it into sublime and simplified it

{% code lineNumbers="true" %}
```
int main(void)

{
  int i;
  char input [72];

  setup();
  printf("Enter your flag: ");
  read(1,input,64);
  i = 0;
  do {
    if (47 < i) {
      puts("Correct!");
label:
      return 0;
    }
    if ((char)~flag[(int)i] != input[(int)i]) {
      puts("Nope.");
      goto label;
    }
    i = i + 1;
  } while( true );
}
```
{% endcode %}

Things learned from the code:

1. Initial variables are declared
2. setup() is called -> no idea so far what it does
3. read() function is used to read STDIN till 64 characters
4. A loop begins where i is currently 0 so 1st if condition is false.
5. Next if condition is encountered. If that is also false, goto makes the statement go to it's label where return 0 is there.
6. Conclusion: Only 1 input is supposed to happen where a correct flag is supposed to be entered.
7. Also, 2nd if condition on line 17 is the key



I inspected setup() and nothing special is happening

<figure><img src="../.gitbook/assets/image (1) (1).png" alt=""><figcaption></figcaption></figure>

So, line 17 must be the key. Let's understand what is happening in English

if ((char)\~flag\[(int)i] != input\[(int)i])

Translation: if the character encoded NOT value of the array flag\[i] where i is the current iteration, is not equal to the input\[i] then display "Nope"

But if it is, continue iteration until 48 correct inputs are given. If 47 inputs are correct display "Correct!"

So, we have to input correct flag array input by input

Another way to understand this translation: If (char)NOT flag\[i] ==input\[i], value is correct



So, let's inspect this flag array

<figure><img src="../.gitbook/assets/image (2) (1).png" alt=""><figcaption></figcaption></figure>

This means, we have to pick up all these characters, bitwise NOT them and then input it in the program

So, first character is 99 in hex. Which means NOT(99) = 66. So on. I utilized ChatGPT to write me a code.

```
#include <stdio.h>
#include <stdint.h>

int main() {
    // The provided hex string
    uint8_t flag_bytes[] = {
        0x99, 0x93, 0x9e, 0x98, 0x84, 0x93, 0xcf, 0xcf,
        0x8f, 0x8c, 0xa0, 0x9e, 0x91, 0x9b, 0xa0, 0x87,
        0xcf, 0x8d, 0x8c, 0xa0, 0x9e, 0x91, 0x9b, 0xa0,
        0x8d, 0x9a, 0x9e, 0x9b, 0x8c, 0xa0, 0x90, 0xa0,
        0x92, 0x86, 0xa0, 0x9d, 0xc9, 0x99, 0xc8, 0x9a,
        0xcb, 0x99, 0xcd, 0xc6, 0x9c, 0x9a, 0xca, 0x82
    };
    
    size_t length = sizeof(flag_bytes) / sizeof(flag_bytes[0]);

    printf("Original: ");
    for (size_t i = 0; i < length; i++) {
        printf("%02X ", flag_bytes[i]);
    }
    printf("\n");

    printf("Bitwise NOT: ");
    for (size_t i = 0; i < length; i++) {
        uint8_t not_byte = ~flag_bytes[i];
        printf("%02X ", not_byte);
    }
    printf("\n");

    return 0;
}
```

<figure><img src="../.gitbook/assets/image (3) (1).png" alt=""><figcaption></figcaption></figure>

Let's translate this hex in ASCII: [https://www.rapidtables.com/convert/number/hex-to-ascii.html](https://www.rapidtables.com/convert/number/hex-to-ascii.html)

<figure><img src="../.gitbook/assets/image (4).png" alt=""><figcaption></figcaption></figure>

Found the flag. EZ through manual execution

