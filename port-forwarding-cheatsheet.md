# Port Forwarding Cheatsheet

## 1. **Container port forwarding to local system using SSH**

A service running on docker was discovered

![](<.gitbook/assets/image (56).png>)

![](<.gitbook/assets/image (63).png>)

**`ssh -L <local port>:<ip of container>:<remote port> <username>@<host>`**

```
ssh -L 6767:172.17.0.2:8080 aubreanna@internal.thm
```

![](<.gitbook/assets/image (125).png>)

## 2. SSH tunneling / Pivoting

Format:

here,

3 systems are there: attacker,&#x20;

compromised PC,&#x20;

system to go to = victim

here SSH is running on port 443 there fore -p 443. Else supply the port where SSH is running on

Therefore,

ssh -i \<id\_rsa of compromised PC> -p 443 \<hostname of compromissed PC>@\<ip of compromised PC> -L \<local port to forward on>:\<victim IP>:\<victim port to forward to on our local system>



ssh -i ssh\_key -p 443 root@172.16.1.1 -L 8080:172.16.1.2:22

<figure><img src=".gitbook/assets/image (3) (2).png" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/image (4).png" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/image (5) (1).png" alt=""><figcaption></figcaption></figure>

## 3. Proxychains nmap scan to a hidden PC accessible via pivoting

Premise:&#x20;

Attacker machine: 10.10.0.10

Compromised machine: 10.10.0.66, 172.16.1.1

Victim Machine: 172.16.1.2



Target: nmap scan 172.16.1.2

Process: Use ssh -D option to create a proxy on local port. Here, 8080. 443 is the port where victim SSH is running.

ssh -D 127.0.0.1:8080 gibson@172.16.1.1 -p 443

<figure><img src=".gitbook/assets/image (9).png" alt=""><figcaption></figcaption></figure>

Then add this in proxychains conf file

<figure><img src=".gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>

Then run nmap scan

<figure><img src=".gitbook/assets/image (5).png" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/image (8).png" alt=""><figcaption></figcaption></figure>
