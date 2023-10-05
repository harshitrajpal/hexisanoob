# Reverse Engineering

Some basic reverse engineering and assembly comprehension as I learn it.

If you think my understanding or explanation is wrong somewhere, leave a comment and I'll review/update the page.



### Basics of Ghidra

<figure><img src="../.gitbook/assets/image (221).png" alt=""><figcaption></figcaption></figure>

This is how Ghidra looks.

1. Decompiler -> By understanding the assembly, ghidra tries to recreate some of the logic of the code. Decompiler will give a better understanding of the underlying code.
2. Program stack -> Disassembled code
3.  Program tree is a logical segmentation of our code.

    1. .data => Where program's global variables live
    2. .rodata => read only data lives here including printf statements
    3. .bss => uninitialized global variables
    4. .eh\_frame and .eh\_frame\_hdr => Example of code specific tree. These two are specifically for code in C/C++ which handles excecptions. When an exception occurs, it has to do unwinding of the stack where the exception can be caught.
    5. .init and .fini => For constructors and deestructors.
    6. .note.gnu.build-id => Compiler puts this in when compiled, so you can tell the difference between two separate compiled versions.

    What loader does: These trees get loaded in the memory during execution. Loader checks executable header and checks what sections there are and then finally plot them in the memory. Finally, it starts execution.



