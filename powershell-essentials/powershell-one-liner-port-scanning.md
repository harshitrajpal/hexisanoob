# Powershell One Liner Port Scanning

20..25 | %{(Test-NetConnection -ComputerName 10.10.0.5 -Port $\_ | Select @{Name="Port Number"; Expression = {$_.RemotePort\}},@{Name="Port Open"; Expression = {$_.TcpTestSucceeded\}})} 3>$null





Here, Port 20..25 range is scanned. Replace with suitable port range
