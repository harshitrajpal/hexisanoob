# global directive

At the start of any assembly program, the global directive tells the assembler where the program starts from. By default, the assembler takes the label "\_start" as the entry point.



However, if we want, we can define any other entry point as well. For example, here, I've dedfined the entry point "yolo." Here is how GDB breaks the binary's assembly:

<figure><img src="../.gitbook/assets/image (405).png" alt=""><figcaption></figcaption></figure>

So, we can see that the program's first instruction is xor rax,rax which is in the yolo label. I defined this label using "global" as the start point.



Further, I'll now remove the "global" directive and see how gdb takes it.

<figure><img src="../.gitbook/assets/image (406).png" alt=""><figcaption></figcaption></figure>



So, the entry point is still "\_yolo" since this is the first instruction that the assembler encounters.

Now, I explicitly define global \__start. Upon inspection, using "info files" command, I can find the entry point's memory address which is the_ start label

<figure><img src="../.gitbook/assets/image (407).png" alt=""><figcaption></figcaption></figure>

And if it is just \_start and no other label before that, assembler defaults to that.

<figure><img src="../.gitbook/assets/image (410).png" alt=""><figcaption></figcaption></figure>



I also experimented by changing this label's name from \_start as well to a custom name. Seeems like assembler is picking up whatever the first label is in .text section as the entry point

<figure><img src="../.gitbook/assets/image (411).png" alt=""><figcaption></figcaption></figure>



To test this, I added another symbol before \_yolo in .text

<figure><img src="../.gitbook/assets/image (412).png" alt=""><figcaption></figcaption></figure>

