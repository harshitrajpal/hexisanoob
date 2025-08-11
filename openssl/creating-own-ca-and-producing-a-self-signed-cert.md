# Creating own CA and producing a self signed cert

1. Create a private key. `my-ca.key`

```
openssl genrsa -out my-ca.key 2048
```

2. Create the CA now. `my-ca.crt`

```
openssl req -x509 -new -nodes -key my-ca.key -sha256 -days 30 -out my-ca.crt -subj "/CN=SomeAuthority/C=US/O=Amazon.com, Inc/OU=Trust and Privacy"
```

3. Create a CSR config file. Make a new `csr.conf` Add necessary details. These details can be extracted from the browser. Go to the website->See the certificate and figure out all the req and dn and DNS details in this CSR.

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

4. Here I am using the private key generated earlier to also sign cert. This is like master key. But can use a new one too.
5. Using this csr.conf, create a new certificate signing request (CSR). `my-ca.csr`

`openssl req -new -key my-ca.key -out my-ca.csr -config csr.conf`

5. Sign the certificate and obtain self signed SSL. `my-domain.crt`

`openssl x509 -req -in my-ca.csr -CA my-ca.crt -CAkey my-ca.key -CAcreateserial -out my-domain.crt -days 30 -sha256`
