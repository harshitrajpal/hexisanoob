# Numbers and Inputs

Filename: numerix (elf64)

Decompilation gives us this:

<figure><img src="../.gitbook/assets/image (382).png" alt=""><figcaption></figcaption></figure>

Simplifying this in C, something like this comes up:

{% code lineNumbers="true" %}
```
int main(EVP_PKEY_CTX *param_1)

{
  int a;
  uint b;
  long c;
  int d;
  
  init(param_1);
  puts("HEY!! I forgot my favorite numbers...");
  puts("Can you get them from my diary?");
  puts("What\'s my favoritest number?");
  c = get_number();
  if (c == 0xdeadbeef) {
    puts("What\'s my second most favorite number?");
    a = get_number();
    if (a == 0x539) {
      puts("Ok, you\'re pretty smart! What\'s the next one?");
      c = get_number();
      if (c == 0xc0def001337beef) {
        puts("YEAAAAAAAAAH you\'re doing GREAT! One more!");
        b = get_number();
        if ((b & 0xf0f0f0f0) == 0xd0d0f0c0) {
          puts("Awwwwww yeah! You did it!");
          print_flag();
          d = 0;
        }
        else {
          puts("Darn, so close too...");
          d = 1;
        }
      }
      else {
        puts("Ugh, ok, listen, you really need to hit the books...");
        d = 1;
      }
    }
    else {
      puts("What? NO! Try again!!");
      d = 1;
    }
  }
  else {
    puts("No! No! No! That\'s not right!");
    d = 1;
  }
  return d;
}


```
{% endcode %}

Things understood:

1. Program prompts for 4 inputs through a function "get\_number()"
2. Guess all 4 correctly to reveal flag through print\_flag()
3. BUT WAIT. All the numbers are already on the screen in hex. Fishy.
4. get\_number()'s functionality is unknown

Let's inspect get\_number()

<figure><img src="../.gitbook/assets/image (383).png" alt=""><figcaption></figcaption></figure>

So it is taking up a string of max 128 (hex 0x80) characters as input and then using strtol() to convert the input into something. Interesting.

strtol(input,(char \*\*)0x0,10) -> return type long int.

This line means that whatever the input is in string, just extract the number part of it, send the string part to a null pointer (discard it)

For Example: see how strtol works on a string that has an integer and some characters.

<figure><img src="../.gitbook/assets/image (384).png" alt=""><figcaption></figcaption></figure>

So, cool, get\_number would just help us input our numbers.

Line 15 compares the input with "0xdeadbeef". Using python to convert this in integer:

<figure><img src="../.gitbook/assets/image (385).png" alt=""><figcaption></figcaption></figure>

Lets try to input this in the program and confirm if we are on the right track or not

<figure><img src="../.gitbook/assets/image (386).png" alt=""><figcaption></figcaption></figure>

Yes!  We are on the right track. Similarly converting next 2 inputs we find: 1337,  868613086753832687

Finally, whatever I  input would be bitwise AND with 0xf0f0f0f0 and that should be equal to d0d0f0c0

So,

B AND f0f0f0f0 = d0d0f0c0

Mathematically,

B AND f0f0f0f0 AND f0f0f0f0 = d0d0f0c0 AND f0f0f0f0

Therefore,

B AND 1 = d0d0f0c0 AND f0f0f0f0

As per the truth table of AND, 1 AND 1 is 1 and rest all is 0

So, let's say B is 010101010101

B AND 1 = 010101010101 AND 1111111111111 still remains 01010101010101

Thus, B AND 1 is B



So, B is d0d0f0c0 AND f0f0f0f0 in decimal is 3503354048

<figure><img src="../.gitbook/assets/image (387).png" alt=""><figcaption></figcaption></figure>

Inputting these we GET!!!

<figure><img src="../.gitbook/assets/image (390).png" alt=""><figcaption></figcaption></figure>

