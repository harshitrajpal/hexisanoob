# My relevant GDB cheatsheet

`set disassembly-flavor intel`

`disas main`

`break *main`

`break *(address)`

`continue` -> continues execution after the breakpoint

`run`

`stepi` ->run the next instruction

`x/i $pc` -> x is for examine. i is for instruction. $pc means program counter. So the command means examine the instruction at current program counter



`set disassemble-next-line on`

`stepi`



It will do that automatically. Keep hitting enter and it will step through the program!

`info registers` ->Print registers at the current registers



`p $rdi` -> p will print a particular register specified by $sign. here, rdi. THis would be in decimal

`p/x $rdi` -> prints rdi in hex



Now let's say your program hits a function call. Using stepi will step into the function and take you step by step through the function code. SOmetimes this is what we want!

Sometimes, we don't care about what a function is doing. So, we use nexti



`nexti` -> When a function is hit, and nexti is entered, rather than going to the next instruction CPU executes, it goes to the one right after this function call in main



## ARM CPU

Use&#x20;

**ROSETTA\_DEBUGSERVER\_PORT=1234 ./binary**

Then a new terminal:

`gdb-multiarch`

`set architecture i386:x86-64`

`target remote localhost:1234`
