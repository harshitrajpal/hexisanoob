---
description: Steganography
---

# Hidden Message

This was the simplest challenge. I ran exiftool on the photo provided in the challenge

<figure><img src="../../.gitbook/assets/image (202).png" alt=""><figcaption></figcaption></figure>



I saw a base64 encoded message in comment's section. I decoded this like this:

`echo "QW1hem9ue1N0M24wZ3JAcGh5XzEkX2hAcmR9" | base64 -d`

<figure><img src="../../.gitbook/assets/image (203).png" alt=""><figcaption></figcaption></figure>

