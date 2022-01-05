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

