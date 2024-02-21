# Stripping readable function names

**new\_basic.c**

**gcc new\_basic.c -o new\_basic**

```c
#include<stdlib.h>
#include<stdio.h>

void func(){

        printf("Hello. A is less than B");
}

int main()
{

        int a=10;
        int b=20;
        if(a>b)
        {
                func();
        }
        return 0;
}
```



When we compile this, we can see a bunch of readable function names in GDB like main, func.

<figure><img src="../../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

**strip new\_basic**

This will take the binary and remove all of those readable names making it more complicated to reverse engineer.

<figure><img src="../../.gitbook/assets/image (2) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>



### How to overcome this?

**break \_start**

\_start is a special symbol that can be used to make debugger run from the first step even if debugger knows nothing about the program. Unfortunately this isn't the start of the code sincec OS does a lot of library setup before even reaching to the code. But it's a good point to start debugging!
