# Foundational Technology

Cloud works on virtualization as a service and offers a simple pay as you go model charging users for launching instances (virtual machines), storage, processors used on cloud.

Amazon first launched web service sin 2004 (SQS) that implemented grid computing. Then further launched EC2 and S3 in 2006. They tweaked it later which eventually became the cloud that we see today.

Types of cloud: Private, Public, Hybrid ( private for a certain part of company, rest public) and Community(built and architected for specific use like in education).

## Cloud Actors

1. Consumer
2. Provider
3. Auditor
4. Broker
5. Carrier

## Regions and Availability Zones

Regions => A particular infrastructure is launched in regions. There are currently 26 regions with AWS like US EAST, EU, APAC etc.

Instances (VM) can be assigned to availability zones. Availability zones comes within a region. To ensure redundant servers for consumers currently 84 Availability zones are there with AWS.

## Basics of Cloud

AWS launched in SQS in 2004, S3 in 2006, EC2 in 20006. These service started the notion of “Always available on the internet” which eventually gave rise to cloud computing. Definition: According to NIST, Cloud Computing is “ a model for enabling a ubiquitous, convenient , on-demand, network access to a shared pool of configurable computing resources. Eg: network, servers, storage, applications and services that can be rapidly provisioned and released with minimal management or effort or service provider interaction.

Benefits:

1. Pay as you go model - Shift from Capex to Apex (capital Expenditure to Operational Expenditure). Instead of buying a bunch of servers, we are just renting servers/resources
2. Benefit from economies of scale - you’re not buying servers in bulk like the cloud providers are
3. Scalable, elastic - use as much or as little as you need and get billed accordingly.
4. Increased speed and agility.
5. Global expansion within minutes.

## 5 essential characteristics of cloud computing:

1. ON demand self service
2. Broad network access
3. Resource pooling
4. Rapid elasticity - scale up/down rapidly
5. Measured service - Service is sold based on compute time, bits etc.

<figure><img src="../.gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>

## Service models on Cloud

1. SaaS - Software as a service - eg: Zoom. The capability provided to the consumer is to use the provider’s applications running on a cloud infrastructure.
2. Platform as a Service - eg: heroic. The capability enables developers to build, run and operate applications entirely in the cloud without directly provisioning infrastructure components.
3. Infrastructure as a service SaaS - eg: AWS, GCP, Azure - The capability provided to the consumer is to provision processing, storage, networks and other fundamental computing resources where the consumer is able to deploy and run arbitrary software, which can include operating systems and applications. How much the customer vs provider own in these models? 

<figure><img src="../.gitbook/assets/image (3).png" alt=""><figcaption></figcaption></figure>

Cloud deployment models:&#x20;

1. Private - operated and maintained by an organization for internal use. Eg: VMWare or Openstack deployed in an enterprise data center.
2. Public - Offered on the internet and available to multiple consumers. Eg: AWS, Azure, GCP
3. Hybrid - Combines Private and Public so that the compute workload can “burst” to the public cloud as needed. Workloads can run on either the private or public cloud.
4. Community - Built and architected for a specific group and use case. Eg: Google Apps for Education, AWS for federal government.

## Some basic AWS Tools introduction

S3 - Simple Storage System

Stores large objects that may have access permissions Used in cloud backup services like Jungle Disk, Netflix etc.

99.99999999999% durability

In s3 you have a bucket - volumes in a filesystem. Bucket names need to be globally unique You also have objects - named items stored in an S3 bucket. Objects also have to be uniquely named (within the bucket only but - similar to filesystem in windows) Limit - 1 byte to 5 TB. Default limit of 100 buckets per account

Objects in S3 can be accessed using REST API, direct Url and AWS console. You can also use Amazon SDk to programmatically access S3 using python/node.js

Access Control in S3 By default all s3 resources configuration are private and can only be accessed by a resource owner. Permissions are assigned through:

1. ACL - Uses XML syntax to grant access to specific S3 buckets or objects
2. User Policies: Uses AWS IAM policy syntax to grant access to IAM users in your account
3. Bucket policies: Uses the AWS IAM policies syntax to manage access for a particular S3 bucket

ACLs are not recommended to be used today

Bucket and User policies are in JSON format. Eg:&#x20;

```
{
	“Version”: “2012-10-17”,
	“Id”: “ExamplePolicy01”,
	“Statement”: [
		{
			“Sid”: “ExampleStatement01”,
			“Effect”: “Allow”,
			“Principal”: {
				“AWS”: “arn:aws:iam::123456789012:user/Dave”
			},
			“Action”: [
				“s3:GetObject”,
				“s3:GetbucketLocation”,
				“s3:ListBucket”
			],
			“Resource”: [
				“arn:aws:s3:::awsexamplebucket1/*”,
				“arn:aws:s3:::awsexamplebucket”
			]
		}
	]
}
```

Here, ARN stands for Amazon Resource Name. Pretty much everything like users, buckets, log groups in AWS has an ARN. Action = API calls that Dave is allowed to call. Only these actions can be performed by Dave. Resources = The resources on which Dave can perform these actions

These policies can be made a little more granular (finely controlled) by using condition keys

Condition Keys: The accèss policy enables you to specify conditions when granting permissions using the condition element. Here we can use Boolean operators (=, <, > etc) to match your condition against values in the request. For example, when granting a user permission to upload an object, the bucket owner can require that the object be publicly readable by adding the StringEquals condition, as shown below:

<figure><img src="../.gitbook/assets/image (6).png" alt=""><figcaption></figcaption></figure>

When to use Bucket Policy? => If an AWS account that owns a bucket wants to grant permission to users in its account, it can use either. Or you want to manage cross account permissions for all Amazon S3 permissions. Both bucket and user policies support granting permission for all Amazon S2 operations.

When to use User Policy=> In general you can use either a user policy or a bucket policy to manage permissions,. It is a good practice to enforce S3 resource based policies when the bucket require s more security consideration and not only use principal based policies.

“Block Public Access” => A setting for S3 where S3 can not be accessed over the internet. Highly recommended if you’re not using a web server,

\================

Amazon EC2 Virtual machines are known as instances. Amazon EC2 is a platform where one can easily launch virtual machines over the cloud. Fine granular controls are also available like public/private key pair to SSH into instances. It is typically setup with temporary storage that is deleted when you stop or terminate an instance, known as instance store volumes.

Long term data should only be stored in S3 or Ems if it is processed locally. Amazon Machine Images (AMI)

It is sort of a disk image(iso) which you’ll install in your EC2. It contains the OS, applications and other data. Amazon provides some generic ones like Amazon Linux, Fedora Core, Windows Server. You can also deploy your own AMIs but don’t leave any secrets on an image.

Instance types On-demand instances  ￼

Reserved instances - If you can predict the capacity you’ll need, you can use reserved instances for one time reservation fee for purchase for 1 or 3 years Spot instances - Here, we can bid for available capacity. Instance continues until terminated or price rises over bid amount.

\=============

Security Groups

Think of them like a firewall rules. They help with these:

1. Can be applied to groups of EC2 instances.and they tell like what traffic can be allowed in and out from the the machine
2. Each rule specifies aprocool port number and IP addresses that are allowed to reach an instance
3. Only traffic matching one of the rules is allowed through EG: while setting web server, in security groups allow port 80

\=============

Elastic Block Store

Hard Drives sitting inside your server. Unlike local instance store, data stored in EBS is not lost when instance fails or is terminated. Typically EBS is used in web servers

Volumes- EBS storage is allocated in volumes (virtual disk) sizing from 1GB to 1 TB They are placed in specify availability zones only. Be sure to place them near instances otherwise you can’t attach them to instances. Also a single instance can have multiple volumes.

EC2 with EBS roots

1. EC2 instances can mount an EBS volume
2. Result: instance data persist after terminating and restarting, similar to suspending and resuming a laptop
3. You can Alsop create snapshots of volumes. Useful for forensics etc.

\=============

AWS console demo: [https://aws.amazon.com/console/](https://aws.amazon.com/console/)

1. If you created a resource in a particular region and then you switch it, it won’t be visible on the console.
2. A resource which is created in Global region will be accessible everywhere.
