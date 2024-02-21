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
