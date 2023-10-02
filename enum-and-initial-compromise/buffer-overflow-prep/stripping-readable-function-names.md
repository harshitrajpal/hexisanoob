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

**strip new\_basic**

This will take the binary and remove all of those readable names making it more complicated to reverse engineer.

