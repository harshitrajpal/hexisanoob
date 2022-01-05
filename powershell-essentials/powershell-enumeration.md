# Powershell Enumeration

1. **Get-LocalUser** : Enumerate local users
2. **Get-LocalGroup** : Enumerate groups
3. **Get-LocalUser -SID "S-1-5-21-1394777289-3961777894-1791813945-501"** : Find the user to which the defined SID belongs
4. **Get-LocalUser | Where-Object -Property PasswordRequired -Match false** : to find users that have their password required values set to False
5. **Get-NetIPAddress** : Find IP address info
6. **Get-NetTCPConnection** : Command to find about current TCP connections
7. **Get-NetTCPConnection | Where-Object -Property State -Match Listen** : Currently listening ports
8. **Get-Hotfix** : Patch update information
9. **Get-ChildItem -Path C:\ -Include \*.bak\* -File -Recurse -ErrorAction SilentlyContinue** : Finding files (here, backup file)
10. **Get-ChildItem C:\* -Recurse | Select-String -pattern API**_**KEY** : Find files containing the string "API\_KEY" (Similar to Grep)_
11. **Get-Process** : List all processes
12. **Get-ScheduleTask** : Find scheduled task
13. **Get-Acl \<path>** : Find ownership info
14. **Get-Content -Path 'C:\Users\dark\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadline\ConsoleHost\_history.txt' :** Just like Linux bash, Windows powershell saves all previous commands into a file called ConsoleHost\_history. This is located at %userprofile%\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadline\ConsoleHost\_history.txt
