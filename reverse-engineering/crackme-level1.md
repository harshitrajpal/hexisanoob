# Crackme: level1

Author: nima



Decompiled:

<figure><img src="../.gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>

initial run:&#x20;

<figure><img src="../.gitbook/assets/image (1) (1).png" alt=""><figcaption></figcaption></figure>

I simplified this code:



```
undefined8 main(int argc,char **argv)

{
  char *var;
  
  if (argc == 2) {
    var = *argv; //equal to var = argv[0] or the "./level" argument. Subsequently argv[1] argvecomes paramter input
    if ((int)*var + (int)*argv[1] == 0x6e) {
      if ((((var[3] == 'n') || (var[4] == 'i')) || (var[5] == 'm')) || (var[6] == 'a'))
      {
        test(var[3],*argv[1]);
      }
      else {
        printf("Try more .... 3/5");
      }
    }
    else {
      printf("Try More ....2/5");
    }
  }
  else {
    printf("Try more ... 1/5");
  }
  return 0;
}
```



Upon inspecting Ghidra, we observe these things mainly:

1. &#x20;Coded is taking in arguments
2. First condition is that arguments should be 2. So ideally, ./level1 \<argument>
3. var = \*argv actually means that var is argv\[0] which is  the name of the program run "./level1"
4. (int)\*var -> ASCII of the first character of var (which is "." (period)) + \*argv\[1] should be equal to 110 (decimal for 0x6e).
5. period (".") is 46 in ASCII, so argv\[1]  should begin with "@" (64 in decimal). totalling 110
6. Either var\[3] or var\[4] or var\[5] or var\[6] should meet condition. Let's make var\[6] true. So program name now becomes ./levea1
7. Finally it reaches test()

<figure><img src="../.gitbook/assets/image (2) (1).png" alt=""><figcaption></figcaption></figure>

As it happens, this is already true because our first argument starts with "@"

So, "./levea1 @"

should crack the binary. Let's see



<figure><img src="../.gitbook/assets/image (401).png" alt=""><figcaption></figcaption></figure>



Note: \*\*argv in int main is conceptually a 2D array

So, "./program Hello" command initializes two arrays

argv\[0] -> ./program

argv\[1] -> Hello

In Assembly this goes in RSI.

* `argc` (argument count) is passed in the `RDI` register.
* `argv` (argument vector) is passed in the `RSI` register.

`int main(int argc, char *argv[])`

&#x20;

