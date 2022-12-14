# Powershell Basics

Powershell is the Windows Scripting Language and shell environment that is built using the .NET framework.

Most Powershell commands, called _cmdlets,_ are written in .NET. Unlike other scripting languages and shell environments, the output of these _cmdlets_ are objects - making Powershell somewhat object oriented.

This also means that running cmdlets allows you to perform actions on the output object(which makes it convenient to pass output from one _cmdlet_ to another). The normal format of a _cmdlet_ is represented using **Verb-Noun**; for example the _cmdlet_ to list commands is called `Get-Command.`

Common verbs to use include:

* Get
* Start
* Stop&#x20;
* Read
* Write
* New
* Out

### Get-Help

`Get-Help Command-Name`\
You can also use "-examples" to see how it is used\
`Get-Help Command-Name -Examples`

![](<../.gitbook/assets/image (110).png>)

Get-Command gets all the _cmdlets_ installed on the current Computer. The great thing about this _cmdlet_ is that it allows for pattern matching like the following

`Get-Command Verb-*` or `Get-Command *-Noun`

Running `Get-Command New-*` to view all the _cmdlets_ for the verb new displays the following

![](<../.gitbook/assets/image (5) (1) (1).png>)

The Pipeline(|) is used to pass output from one _cmdlet_ to another. A major difference compared to other shells is that instead of passing text or string to the command after the pipe, powershell passes an object to the next cmdlet

To view these details, pass the output of a _cmdlet_ to the Get-Member _cmdlet_

`Get-Command | Get-Member -MemberType Method`

![](<../.gitbook/assets/image (73).png>)

**Creating Objects From Previous **_**cmdlets**_

One way of manipulating objects is pulling out the properties from the output of a cmdlet and creating a new object. This is done using the `Select-Object` _cmdlet._

`Get-ChildItem | Select-Object -Property Mode, Name`

![](<../.gitbook/assets/image (102).png>)

**Filtering Objects**

When retrieving output objects, you may want to select objects that match a very specific value. You can do this using the `Where-Object` to filter based on the value of properties.

**Get-Service | Where-Object -Property Status -eq stopped**

![](<../.gitbook/assets/image (67).png>)

![](<../.gitbook/assets/image (32).png>)

Other operators than "-eq" can be found [here](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.core/where-object?view=powershell-7.2\&viewFallbackFrom=powershell-6)

**Sort Object**

When a _cmdlet_ outputs a lot of information, you may need to sort it to extract the information more efficiently. You do this by pipe lining the output of a _cmdlet_ to the `Sort-Object` _cmdlet_.

![](<../.gitbook/assets/image (151).png>)

**Finding a file**

`Get-ChildItem -Path c:\ -Include`` `_`*`_`interesting-file.txt`_`*`_` ``-File -Recurse -ErrorAction SilentlyContinue`

![](<../.gitbook/assets/image (42).png>)

**Counting an output (using measure)**

Lets count the cmdlets installed in the system

`Get-Command | Where-Object -Parameter CommandType -eq Cmdlet | measure`

![](<../.gitbook/assets/image (87) (1).png>)

**Getting a file's hash**

We want MD5 hash here, so:

`Get-FileHash -Path "C:\Program Files\interesting-file.txt.txt" -Algorithm MD5`

![](<../.gitbook/assets/image (124).png>)

**Current Working Directory?**

`Get-Location`

![](<../.gitbook/assets/image (22).png>)

**To see if a directory exists?**

`Get-Location -Path "C:\Users\Administrator\Documents\Passwords"`

**Making a request to web server**

`Invoke-WebRequest`

**Base64 decode a file**

`Get-ChildItem -Path C:/ -Include b64.txt -Recurse -File -ErrorAction SIlentlyContinue`

`certutil -decode "path\file.txt" decode.txt`

![](<../.gitbook/assets/image (13).png>)

