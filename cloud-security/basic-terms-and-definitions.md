# Basic terms and definitions

## Amazon S3

Stands for Simple Storage Systems. It stores large objects that may have access permissions.

1. used in cloud backup
2. used in streaming video like Netflix
3. used internally by Amazon to store virtual machines.
4. Provides 99.99999999% durability
5. Provides 99.99% availability.

**Key concepts:**

1. Buckets=> This is the container of storage. Just like Volumes in a physical filesystems, buckets are volumes in cloud. They have globally unique names. Max 100 buckets configurable per account.
2. Objects=> Named items stored within a bucket like pictures, videos etc. 1 byte to 5 TB in any format. We can create unlimited objects in a bucket theoretically.
3. Note: Names within a bucket must be unique. Just like in Windows, 2 files in a same folder can't have the exact same name. This causes indexing problem and is there in buckets too.
4. Objects in a bucket can only be accessed via: REST/SOAP API calls, via URL, via AWS console. Python offers libraries like boto3 which make API calls to objects and can perform functions on them. This can be configured as a lambda function too.

**Access control in S3**

1. by default all Amazon S3 resources configurations are private.
2. only the resource owner can access the resource.
3. The resource owner can optionally grant access permissions for others by writing an access policy. An example policy is as follows. This grants listing of bucket objects and Read/Write on them on a bucket called "example-bucket"

```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "ListObjectsInBucket",
            "Effect": "Allow",
            "Principal": "*",
            "Action": "s3:ListBucket",
            "Resource": "arn:aws:s3:::example-bucket"
        },
        {
            "Sid": "AllObjectActions",
            "Effect": "Allow",
            "Principal": "*",
            "Action": "s3:*Object",
            "Resource": "arn:aws:s3:::example-bucket/*"
        }
    ]
}
```

4. by default, when another AWS account uploads an object to your S3 bucket, that account owns the object to your S3 bucket, has access to it, owns it, and can grant other users access to it via ACLs.

**Permissions in S3**

1. ACL=> Uses XML to grant access to specific S3 buckets or Objects
2. user Policies => Use the AWS IAM policy syntax to control access of a specific IAM user in your account. JSON dedfault.
3. Bucket policies=> Use the AWS IAM policy syntax to control access of a specific S3 bucket in your account. JSON dedfault.

Note: ACLs are not recommended these days because they have finite controls and don't include all S3 permissions.

****

**ARN**: Amazon Resource Name. All users, resources etc have an ARN.
