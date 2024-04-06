# Binary Search

The challenge gave a number guessing game between 1 and 1000 but we only have 10 tries.

By following the logic of binary search manually, we guess 500 (mid point) and then as per higher  or lower prompt, we'll put values in between the current low and high limit (eg, if higher, then 750)

Algorithm I used:

```
guess = 500
current_low =0
current_high = 1000
i = 10

while (i!=0):
    prompt = receive()
    input(guess)
    message = receive()
    if message contains "Congratulations!":
        return guess
        exit
    if message = "higher":
        current_low = guess
        guess = (current_high-current_low)/2 + current_low
    else if message = "lower":
        current_high = guess
        guess = (current_high-current_low)/2 + current_high
        
        
    i=i-1
    
exit
            
```

(Could be wrong technically, but it is the rough idea I drew up)



<figure><img src="../../../.gitbook/assets/image (12).png" alt=""><figcaption></figcaption></figure>

