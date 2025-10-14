---
description: >-
  Start the engagement and use this IP address to identify their AWS account ID
  via a public S3 bucket so we can commence the process of enumeration.
---

# S3 - Identify the AWS Account ID from a Public S3 Bucket

## Real-world context

If threat actors get their hands on an AWS Account ID, they can try to identify the IAM roles and users tied to that account. They can do this by taking advantage of detailed error messages that AWS services return when inputting an incorrect username or role name. These messages can verify if an IAM user or role exists, which can help threat actors compile a list of possible targets in the AWS account. It's also possible to filter public EBS and RDS snapshots by the AWS Account ID that owns it.



After the nmap scan we discovered a website. Upon inspecting the source code we got to know about the s3 bucket being used. But we could not access it.

<figure><img src="../../.gitbook/assets/image (9) (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (10) (1).png" alt=""><figcaption></figcaption></figure>

With the S3 bucket name we can attempt to get the ID of the AWS Account it's hosted in. [Research](https://cloudar.be/awsblog/finding-the-account-id-of-any-public-s3-bucket/) by [Ben Bridts](https://twitter.com/benbridts) revealed that it's possible to quickly brute force the AWS account ID an S3 bucket belongs to. Reading this research post and also reviewing the code [here](https://github.com/WeAreCloudar/s3-account-search/blob/main/s3_account_search/cli.py) is recommended, but a TL;DR is that this script creates policy that utilizes the new `S3:ResourceAccount` Policy Condition Key to evaluate whether to grant us access to an S3 bucket based on the AWS account that the bucket belongs to. Fortunately the script doesn't have to guess a trillion different account IDs to find the right one, the available search space is massively reduced by leveraging string matching and wildcards. Each correctly matched digit is appended to a variable, and the request is repeated until the account ID is found.

<figure><img src="../../.gitbook/assets/image (13) (1).png" alt=""><figcaption></figcaption></figure>



<figure><img src="../../.gitbook/assets/image (11) (1).png" alt=""><figcaption></figcaption></figure>

rebooting terminal worked and I could use s3-account-search binary. But We can also use virtual env to install pip packages to not cause external environment errors. Also, have pip.conf file modified to set external environment variable as true

<figure><img src="../../.gitbook/assets/image (15).png" alt=""><figcaption></figcaption></figure>

^ Here is the account ID.

This reveals the AWS account ID `107513503799`. We can use this information to hunt down public resources that might have been accidently exposed by the account owner, such as public EBS and RDS snapshots.

First, it would be good to know the AWS region that the S3 bucket was created in, as public snapshots are available to all users in the same region that the EBS or RDS snapshot was created in. It's likely that if the S3 bucket was created in a specific region, that other resources will be available there too!

To find the S3 bucket region we can use another trick, this time with cURL.

```
curl -I https://mega-big-tech.s3.amazonaws.com
```

<figure><img src="../../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption><p>us-east-1 is the bucker region in the header as we can see</p></figcaption></figure>

From the account ID and region we can now go to the amazon console in personal account and go to EC2 and look for public snapshots.

<figure><img src="../../.gitbook/assets/image (3) (1) (1).png" alt=""><figcaption></figcaption></figure>

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

the flag was the account ID already found but good to know how to loot public EBS volumes.



