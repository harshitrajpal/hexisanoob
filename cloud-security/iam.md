# IAM

Federation identity - Concept where you don't have to create multiple credentials for multiple different services. Eg: Active Directory. One account can be used for all applications within an organization. Eg: SSO (sign in with google, logiin with facebook etc). SAML and OAUTH2 are used in SSO.

Cloud environments use SSO heavily.

**SAML 2.0 and OAUTH 2.0 components**:

IDP - Identity Proivder - Entity which knows how to authenticate

SP - Service provider - Application which communicates with the IDP in order to obtain authentication information of the user

Client - Web browser used to interact with resources.



SAML - SAML is designed to auth the user by providing user id data to a service

Oauth - Standard authorization protocol permitting user to share access to a specific resource within an SP. You use it to jump from one service to other without inputting credentials everytime.



**Common attacks on OAuth 2.0:**

1. CSRF -&#x20;

<figure><img src="../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

defense techniques: include user interaction, and other CSRF defence techniques talked about here: [https://hexisanoob.gitbook.io/hexisanoob/enum-and-initial-compromise/web-pentest/csrf-mitigation](https://hexisanoob.gitbook.io/hexisanoob/enum-and-initial-compromise/web-pentest/csrf-mitigation)

2. Clickjacking
3. Phishing

## AWS IAM

• AWS Identity and Access Management (IAM) enables you to manage access to AWS services and resources securely. Using IAM, you can create and manage AWS users and groups, and use permissions to allow and deny their access to AWS resources.&#x20;

• IAM enables your users to control access to AWS service APIs and to specific resources. IAM also enables you to add specific conditions such as time of day to control how a user can use AWS, their originating IP address, whether they are using SSL, or whether they have authenticated with a multi-factor authentication device.

• Centralized control of your AWS account&#x20;

• Shared access to your AWS account&#x20;

• Identity federation using AD, Facebook, Linkedin&#x20;

• Multifactor Auth&#x20;

• Temporary access for users and devices&#x20;

• Password rotation policies&#x20;

• PCI DSS compliant

### Components

Users – people&#x20;

Groups – collection of users&#x20;

Roles – Assigned to AWS resources&#x20;

Policies – JSON document which defines permissions. Sample Policy: [https://docs.aws.amazon.com/AmazonS3/latest/userguide/example-policies-s3.html#iam-policy-ex0](https://docs.aws.amazon.com/AmazonS3/latest/userguide/example-policies-s3.html#iam-policy-ex0)



Go to AWS console -> IAM -> Users -> Create a new user.

