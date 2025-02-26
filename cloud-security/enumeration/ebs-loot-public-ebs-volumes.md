---
description: >-
  Assuming we have the account ID we found through xyz mechanisms. Here, through
  user ID bruteforce by public S3 bucket URL.
---

# EBS - Loot Public EBS Volumes

First, it would be good to know the AWS region that the S3 bucket was created in, as public snapshots are available to all users in the same region that the EBS or RDS snapshot was created in. It's likely that if the S3 bucket was created in a specific region, that other resources will be available there too!

To find the S3 bucket region we can use another trick, this time with cURL.

```
curl -I https://mega-big-tech.s3.amazonaws.com
```

<figure><img src="../../.gitbook/assets/image (1) (1).png" alt=""><figcaption><p>us-east-1 is the bucker region in the header as we can see</p></figcaption></figure>

From the account ID and region we can now go to the amazon console in personal account and go to EC2 and look for public snapshots.

<figure><img src="../../.gitbook/assets/image (3) (1).png" alt=""><figcaption></figcaption></figure>

You can loot public EC2 snapshots: [https://pwnedlabs.io/labs/loot-public-ebs-snapshots](https://pwnedlabs.io/labs/loot-public-ebs-snapshots)

{% embed url="https://medium.com/@willygichohim/loot-public-ebs-snapshots-e809d1b60bf4" %}

<figure><img src="../../.gitbook/assets/image (4) (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (5) (1).png" alt=""><figcaption></figcaption></figure>

Then launch an EC2 instance and attach this volume

<figure><img src="../../.gitbook/assets/image (6) (1).png" alt=""><figcaption></figcaption></figure>



You can SSH into the EC2 then and run lsblk command

<figure><img src="../../.gitbook/assets/image (7) (1).png" alt=""><figcaption></figcaption></figure>

Here, as we specified earlier xvdf1 is our disk. let's mount this

<figure><img src="../../.gitbook/assets/image (8) (1).png" alt=""><figcaption></figcaption></figure>

