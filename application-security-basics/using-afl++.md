---
description: >-
  A tool to fuzz applications, find code coverage, and discover crashes in the
  application
---

# Using AFL++

git clone https://github.com/AFLplusplus/AFLplusplus.git

make distrib

sudo make install



Make the application using afl-gcc. Mandatory! This removes instrumentation on binary and lets AFL add additional code needed to understand inputs required.

afl-gcc giftcardreader.c -o giftcardreader

afl-fuzz -i aflfuzz/inputs -o aflfuzz/outputs -- ./giftcardreader 1 @@

Now I put all the testcases that I need to supply to the application in aflfuzz/inputs directory and create an empty directory aflfuzz/outputs in which AFL shall put its output testcases.

<figure><img src="../.gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../.gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../.gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>
