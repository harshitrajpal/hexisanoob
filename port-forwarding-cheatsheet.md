# Port Forwarding Cheatsheet

1. **Container port forwarding to local system using SSH**

A service running on docker was discovered

![](<.gitbook/assets/image (56).png>)

![](<.gitbook/assets/image (63).png>)

**`ssh -L <local port>:<ip of container>:<remote port> <username>@<host>`**

```
ssh -L 6767:172.17.0.2:8080 aubreanna@internal.thm
```

![](<.gitbook/assets/image (125).png>)

2\. SSH tunneling / Pivoting

Format:

here,

3 systems are there: attacker,&#x20;

compromised PC,&#x20;

system to go to = victim

here SSH is running on port 443 there fore -p 443. Else supply the port where SSH is running on

Therefore,

ssh -i \<id\_rsa of compromised PC> -p 443 \<hostname of compromissed PC>@\<ip of compromised PC> -L \<local port to forward on>:\<victim IP>:\<victim port to forward to on our local system>



ssh -i ssh\_key -p 443 root@172.16.1.1 -L 8080:172.16.1.2:22

<figure><img src=".gitbook/assets/image (3).png" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/image (5).png" alt=""><figcaption></figcaption></figure>
