# Breakpoints

We can hit breakpoints at absolute addresses:

break \*\<address>

Example, break \*0000000000400da



We can also hit breakpoints at function names. Example,

break strcmp

break \_start



And we can also hit breakpoints at relative offsets of RIP. For example, if currently RIP is at the start,

break \*$rip+42



This would set a breakpoiint at 42nd offset from the starting point.



Finally, if debugging symbols are ON in a binary, we can see the code using "list" command. So we can also set a breakpoint at an absolute line in the code. For example,

break 10

Sets a breakpoint at line 10 of the code (As seen on the last page where strcmp was there)
