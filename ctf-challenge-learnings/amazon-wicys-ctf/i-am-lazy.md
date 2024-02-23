# I am Lazy

The challenge presented us with a 47MB zip file called "encoded\_output.txt"

<figure><img src="../../.gitbook/assets/image (2) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Upon opening the file I noticed that it was encoded using some cipher.

<figure><img src="../../.gitbook/assets/image (2) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Now, we had to analyze which encryption it was using. Using dcode.fr I saw a couple of possibilities. I didn't want to use the lesser known ciphers. So I stuck with base64. But since flag is Amazon{......} no way the base64 cipher could encoded such a small string to an 80MB text file.

So, I created a simple bash one liner to decipher it. I didn't know how many times it was encoded, so an educated guess was to run the script 1000 times. The script is simply running cat on the file, searching for string starting with "Amazon" and if not found, inputting the first time decoded text into the loop again and again until we find the flag.



`data=$(cat encoded_output.txt); for i in {1..1000}; do data=$(echo "$data" | base64 -d); [[ $data == Amazon* ]] && echo "$data" && break; done`



<figure><img src="../../.gitbook/assets/image (212).png" alt=""><figcaption></figcaption></figure>

