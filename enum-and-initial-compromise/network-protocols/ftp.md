# FTP

### Default Configuration

One of the most used FTP servers on Linux-based distributions is [vsFTPd](https://security.appspot.com/vsftpd.html). The default configuration of vsFTPd can be found in `/etc/vsftpd.conf`, and some settings are already predefined by default. It is highly recommended to install the vsFTPd server on a VM and have a closer look at this configuration.

**Install vsFTPd**

```shell-session
sudo apt install vsftpd 
```

The vsFTPd server is only one of a few FTP servers available to us. There are many different alternatives to it, which also bring, among other things, many more functions and configuration options with them. We will use the vsFTPd server because it is an excellent way to show the configuration possibilities of an FTP server in a simple and easy-to-understand way without going into the details of the man pages. If we look at the configuration file of vsFTPd, we will see many options and settings that are either commented or commented out. However, the configuration file does not contain all possible settings that can be made.

In the following example, we can see that if the `hide_ids=YES` setting is present, the UID and GUID representation of the service will be overwritten, making it more difficult for us to identify with which rights these files are written and uploaded.

**Hiding IDs - YES**

```shell-session
ftp> ls

---> TYPE A
200 Switching to ASCII mode.
ftp: setsockopt (ignored): Permission denied
---> PORT 10,10,14,4,223,101
200 PORT command successful. Consider using PASV.
---> LIST
150 Here comes the directory listing.
-rw-rw-r--    1 ftp     ftp      8138592 Sep 14 16:54 Calender.pptx
drwxrwxr-x    2 ftp     ftp         4096 Sep 14 17:03 Clients
drwxrwxr-x    2 ftp     ftp         4096 Sep 14 16:50 Documents
drwxrwxr-x    2 ftp     ftp         4096 Sep 14 16:50 Employees
-rw-rw-r--    1 ftp     ftp           41 Sep 14 16:45 Important Notes.txt
-rw-------    1 ftp     ftp            0 Sep 15 14:57 testupload.txt
226 Directory send OK.
```

This setting is a security feature to prevent local usernames from being revealed. With the usernames, we could attack the services like FTP and SSH and many others with a brute-force attack in theory. However, in reality, [fail2ban](https://en.wikipedia.org/wiki/Fail2ban) solutions are now a standard implementation of any infrastructure that logs the IP address and blocks all access to the infrastructure after a certain number of failed login attempts.

Another helpful setting we can use for our purposes is the `ls_recurse_enable=YES`. This is often set on the vsFTPd server to have a better overview of the FTP directory structure, as it allows us to see all the visible content at once.

**Cheatsheet**

**Download all files**

```shell-session
wget -m --no-passive ftp://anonymous:anonymous@10.129.14.136
```

Upload file

```shell-session
put testupload.txt
```

Footprinting - nmap

```shell-session
find / -type f -name ftp* 2>/dev/null | grep scripts
```

```shell-session
nmap --script-updatedb
```

```shell-session
nmap -sV -p21 -sC -A 10.129.14.136
```

Interacting with FTP

```shell-session
nc -nv 10.129.14.136 21
ftp <ip>
```













