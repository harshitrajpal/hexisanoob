---
description: >-
  This breaks device trust model and makes app believe it is a legit server
  since signed by a trusted private CA
---

# Spoofed SSL, sign with AWS Private CA, launch on Kube

1. Make a new `csr.conf.` Add necessary details. These details can be extracted from the browser. Go to the website->See the certificate and figure out all the req and dn and DNS details in this CSR.

```
[ req ]
default_bits = 2048
prompt = no
default_md = sha256
req_extensions = req_ext
distinguished_name = dn

[ dn ]
C = US
ST = New Jersey
L = Jersey City
O = Bee Server
OU = Infrastructure
CN = api.honey-staging.com

[ req_ext ]
subjectAltName = @alt_names

[ alt_names ]
DNS.1 = api.honey-staging.com
DNS.2 = *.api.honey-staging.com
DNS.3 = localhost
IP.1 = 127.0.0.1
```

1. make a new private key: openssl genrsa -out api.key 2048
2.  Request a new certificate signing request (CSR):&#x20;

    ```
    openssl req -new -key api.key -out spoofed.csr -config csr.conf
    ```
3. Assume the AWS role that allows access to ACM PCA
4.

    ```
    aws acm-pca issue-certificate \
       --profile assume-cross-acct \
       --certificate-authority-arn arn:aws:acm-pca:us-east-1:XXXXXX:certificate-authority/XXXXXXXXX \
       --csr fileb://spoofed.csr \
       --signing-algorithm "SHA256WITHRSA" \
       --validity Value=30,Type="DAYS" \
       --template-arn arn:aws:acm-pca:::template/EndEntityCertificate/V1
    ```
5. This will issue a new signed certificate arn
6. Get the cert and store locally
7. `aws acm-pca get-certificate`\
   `--certificate-authority-arn arn:aws:acm-pca:us-west-2:XXXXXX:certificate-authority/XXXXX`\
   `--certificate-arn arn:aws:acm-pca:us-west-2:XXXX:certificate-authority/XXXXX/certificate/`**`redactedId`**\
   `--output text > api.crt`
8. Open that api.crt and make sure that the chained certificate we have are in two separate lines. For example:
9.

    <figure><img src="../.gitbook/assets/image (454).png" alt=""><figcaption></figcaption></figure>
10. Once done, upload all this to EC2/server where Kubernetes is running
11. Kubernetes has different pods that do different things. `app-tls` pod generally takes care of TLS handling. This is where communication comes and data decrypts and the backend server running at pods like app-api on like port 3000 take care of the logic. app-tls has SSL certificate installed. Our aim is to replace this with our signed cert app-ingress has config of which app-tls secret to send communicationt to.
12. On Kubernetes:\
    \
    a. Create a new secret: k3s kubectl create secret tls my-valid-tls --key api.key --cert api.crt -n default\
    b. Edit the ingress rule app where app-tls receives connection from and forwards to app-tls. This is where we have configured which app-tls secret to use\
    \
    k3s kubectl edit ingress app-ingress -n default\
    \
    \
    Look for the line that says app-tls (or spec.tls.secretName) and replace with out secret "my-valid-tls"



14. That;s it, your server on Kubernetes will now use your spoofed cert.

***

You can also create your own CA and sign your cert with it.

