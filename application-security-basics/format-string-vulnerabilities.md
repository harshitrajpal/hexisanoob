# Format string vulnerabilities

There are specified format strings in C which if not used correctly in code, can even allow an attacker to read random memory which may contain sensitive information.

Format strings:

<figure><img src="../.gitbook/assets/image (5) (1) (2) (1).png" alt=""><figcaption></figcaption></figure>

Example of an attack:

Vulnerable code:\


<figure><img src="../.gitbook/assets/image (8) (3).png" alt=""><figcaption></figcaption></figure>

Here, in printf command, no format string is specified, so it will take in format of string provided in fgets and print it.

Normal working:\
![](<../.gitbook/assets/image (2) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png>)



### Exploitation

Giving %d  and %x as input.

![](<../.gitbook/assets/image (1) (1) (3).png>)

Explanation: %d: It reads whatever integer value is stored in the current execution stack. Not necessarily have to be value of an integer in the defined program, but just random value in memory that is a result of some other program that has executed before.

%x: reads hexadecimal value in stack.

**This also means now we can read the** [**STACK CANARY**](https://hexisanoob.gitbook.io/hexisanoob/application-security-basics/buffer-overflow-defenses)

Also, in some cases, passwords (by using %s for string):

<figure><img src="../.gitbook/assets/image (3) (1) (2) (1).png" alt=""><figcaption></figcaption></figure>

## Case Study

A while ago, When you kept your Wi-Fi name as %n%n%n%n, it crashed iPhones as they read Wi-Fi names on the page.
