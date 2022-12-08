# Powershell Port Scan in a given CIDR

Here CIDR given is 10.10.0.32/29

I have hardcoded the CIDR in foreach loop which can be replaced by two variables taken in as input from the user or by customizing it.



```
function Port-Scan
{
	write-host "Scanning for IP range 10.10.0.32/29"
    
    write-host "Enter start port: "
    $inputstr1 = read-host
	$port1 = $inputstr1 -as [Int32]

    write-host "Enter end port: "
    $inputstr2 = read-host
	$port2 = $inputstr2 -as [Int32]

    write-host ""
    write-host "Scanning Started..."
    write-host ""

    $ErrorActionPreference= ‘silentlycontinue’

    $counter = 0
    $negcounter = 0

    foreach($range in 33..38)
    {
        write-host "Scanning IP ========= 10.10.0.$range"
        foreach($port in $port1..$port2)
        {
            If(Test-Connection –BufferSize 32 –Count 1 –quiet –ComputerName "10.10.0.$range")
            {
                $socket = new-object System.Net.Sockets.TcpClient("10.10.0.$range", $port)
                If($socket.Connected) 
                { 
                    write-host “   port $port open” -ForegroundColor Green
                    $socket.Close()
                    $counter += 1
                }
                else
                {
                    $negcounter += 1
                }
                
            }
            else
            {
                write-host "   10.10.0.$range is not up!" -ForegroundColor Red
                break
            }
            
        }

        write-host "$counter ports open on host 10.10.0.$range"
        write-host "$negcounter ports closed on host 10.10.0.$range"
        write-host ""
        $counter = 0
        $negcounter = 0
    }

    write-host ""
    write-host "Scanning Ended"
}
```

![](../.gitbook/assets/image.png)
