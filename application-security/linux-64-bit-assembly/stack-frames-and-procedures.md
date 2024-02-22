# Stack-Frames and Procedures

When a procedure is called, we need to create a Stack Frame.

This Stack Frame acts like a wall and isolates all the data created by previous procedures. Typically it's created at the current location of RSP. It is removed when the procedure returns.

Why do we need it? -> If the program isn't touching the stack,  it is not needed

But according to stackovverflow.com: "Stack frames are convenient when you save registers and store local variables in stack - to make writing and debugging easier: you just set `ebp` to a fixed point in stack and address all stack data using `ebp`. And it's easier to restores `esp` at the end.

Also, debuggers often expect stack frames to be present, otherwise you can get inaccurate stack call for example." ([https://stackoverflow.com/questions/17239781/x86-assembly-why-do-i-need-stack-frames](https://stackoverflow.com/questions/17239781/x86-assembly-why-do-i-need-stack-frames))

Here is an example. When I create a function in C to print something:

<figure><img src="../../.gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>

Upon inspecting the disassembly of main we will see a stack frame set up as the first two instructions and then always ends in instructions: leave and ret.

<figure><img src="../../.gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>

We would see in many assembly programs that the instructions:

push rbp\
mov rbp,rsp\
\
Is always given at the start and then they end with

leave\
ret



Leave-> It is basically doing the reverse of push rbp;mov rbp,rsp

that is

leave=>\
mov rsp,rbp\
pop rbp



This is essentially maintaining the stack frame.



