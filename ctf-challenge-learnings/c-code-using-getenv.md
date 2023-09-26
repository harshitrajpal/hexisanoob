# C code using getenv()

In "blog" CTF on TryHackMe, I encountered a chellenge where a C binary had SUID set. The binary's strings and ltrace output looked like this:

<figure><img src="../.gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>

<figure><img src="../.gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>

We see the binary is checking if environment variable "admin" is set.

And in strings output, we see "system" call being made. According to me pseudocode goes something like:

```clike
#include<whatever>
int main(){
    if(getenv("admin"))
    {
        system("/bin/bash");
    }
}
```

So, I set "admin" environment variable and ran the binary called "checker" and got root!

<figure><img src="../.gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>
