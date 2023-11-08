# DNS/Domain Enum Masterguide

## Working

* Address Bar => www.google.com
* Step 1: You make a request to a website. The first thing that your computer does is check its local cache to see if it's already got an IP address stored for the website; if it does, great. If not, it goes to the next stage of the process.
* Assuming the address hasn't already been found, your computer will then send a request to what's known as a _recursive_ DNS server.
* These will automatically be known to the router on your network. Many Internet Service Providers (ISPs) maintain their own recursive servers, but companies such as Google and OpenDNS also control recursive servers. (eg: 8.8.8.8 is for google.com)
* This is how your computer automatically knows where to send the request for information: details for a recursive DNS server are stored in your router. This server will also maintain a cache of results for popular domains; however, if the website you've requested isn't stored in the cache, the recursive server will pass the request on to a _root name_ server.
* Before 2004 there were precisely 13 root name DNS servers in the world.The root name servers essentially keep track of the DNS servers in the next level down, choosing an appropriate one to redirect your request to. These lower level servers are called _Top-Level_ _Domain_ servers.
* Top-Level Domain (TLD) servers are split up into extensions. So, for example, if you were searching for tryhackme**.com** your request would be redirected to a TLD server that handled `.com` domains. If you were searching for bbc**.co.uk** your request would be redirected to a TLD server that handles `.co.uk` domains.
* As with root name servers, TLD servers keep track of the next level down: _Authoritative name servers_. When a TLD server receives your request for information, the server passes it down to an appropriate Authoritative name server.
* Authoritative name servers are used to store DNS records for domains directly. In other words, every domain in the world will have its DNS records stored on an Authoritative name server somewhere or another; they are the source of the information. When your request reaches the authoritative name server for the domain you're querying, it will send the relevant information back to you, allowing your computer to connect to the IP address behind the domain you requested.

## DNS Records (source: cloudflare)

[DNS](https://www.cloudflare.com/learning/dns/what-is-dns/) records (aka zone files) are instructions that live in authoritative [DNS servers](https://www.cloudflare.com/learning/dns/dns-server-types/) and provide information about a domain including what [IP address](https://www.cloudflare.com/learning/dns/glossary/what-is-my-ip-address/) is associated with that domain and how to handle requests for that domain. These records consist of a series of text files written in what is known as DNS syntax. DNS syntax is just a string of characters used as commands that tell the DNS server what to do. All DNS records also have a ‘[TTL](https://www.cloudflare.com/learning/cdn/glossary/time-to-live-ttl/)’, which stands for time-to-live, and indicates how often a DNS server will refresh that record.

* **A record** - The record that holds the IP address of a domain. [Learn more about the A record.](https://www.cloudflare.com/learning/dns/dns-records/dns-a-record/)
* **AAAA record** - The record that contains the IPv6 address for a domain (as opposed to A records, which list the IPv4 address). [Learn more about the AAAA record.](https://www.cloudflare.com/learning/dns/dns-records/dns-aaaa-record/)
* **CNAME record** - Forwards one domain or subdomain to another domain, does NOT provide an IP address. [Learn more about the CNAME record.](https://www.cloudflare.com/learning/dns/dns-records/dns-cname-record/)
* **MX record** - Directs mail to an email server. [Learn more about the MX record.](https://www.cloudflare.com/learning/dns/dns-records/dns-mx-record/)
* **TXT record** - Lets an admin store text notes in the record. These records are often used for email security. [Learn more about the TXT record.](https://www.cloudflare.com/learning/dns/dns-records/dns-txt-record/)
* **NS record** - Stores the name server for a DNS entry. [Learn more about the NS record.](https://www.cloudflare.com/learning/dns/dns-records/dns-ns-record/)
* **SOA record** - Stores admin information about a domain. [Learn more about the SOA record.](https://www.cloudflare.com/learning/dns/dns-records/dns-soa-record/)
* **SRV record** - Specifies a port for specific services. [Learn more about the SRV record.](https://www.cloudflare.com/learning/dns/dns-records/dns-srv-record/)
* **PTR record** - Provides a domain name in reverse-lookups. [Learn more about the PTR record.](https://www.cloudflare.com/learning/dns/dns-records/dns-ptr-record/)

## Enumeration

When you visit a website in your web browser this all happens automatically, but we can also do it manually with a tool called **`dig`** . Like ping and traceroute, dig should be installed automatically on Linux systems.

eg: if we want to query google.com with our custom recursive server we do dig \<url>\<space>@\<recursive-dns-server>

**`dig google.com @1.1.1.1`**

<div align="left">

<img src="../.gitbook/assets/image (108).png" alt="">

</div>

**`TTL =>`** Another interesting piece of information that dig gives us is the TTL (**T**ime **T**o **L**ive) of the queried DNS record. As mentioned previously, when your computer queries a domain name, it stores the results in its local cache. The TTL of the record tells your computer when to stop considering the record as being valid -- i.e. when it should request the data again, rather than relying on the cached copy.

**`ANSWER SECTION =>`** This is the result (resolved IP of the URL as per 1.1.1.1's record)

## Cheatsheet

* Digging IP address: dig google.com
* Digging for short answers: dig google.com +short

<div align="left">

<img src="../.gitbook/assets/image (54).png" alt="">

</div>

* Reverse IP lookup: dig -x 142.250.206.174\


<div align="left">

<img src="../.gitbook/assets/image (31).png" alt="">

</div>

* Query any DNS record: dig google.com ANY\


<div align="left">

<img src="../.gitbook/assets/image (140).png" alt="">

</div>

* dig for particular records: \
  dig hostinger.com txt (Query TXT record)\
  dig hostinger.com cname (Query CNAME record)\
  dig hostinger.com ns (Query NS record)\
  dig hostinger.com A (Query A record)

<div align="left">

<img src="../.gitbook/assets/image (85).png" alt="">

</div>

* Trace DNS path: dig hostinger.com +trace (It will query the name servers starting from the root and subsequently traverses down the namespace tree using iterative queries)\


<div align="left">

<img src="../.gitbook/assets/image (15) (1) (1) (1).png" alt="">

</div>

## DNS Zone Transfer

It is a process of sharing DNS records, the whole zone file, or only the most recent DNS records. The DNS zones are a part of the DNS that can be administrated through an authoritative DNS server. The whole DNS is organized with a hierarchical structure – root level, TLD, domain name, subdomain, etc. There are different levels that can be managed independently. The purpose of the division is exactly to facilitate the administration of the DNS. DNS zones allow exactly this, to manage a partition of the domain namespace. The DNS administrator of a higher level needs to delegate a Master DNS zone to another administrator, so he or she can manage a lower level zone. The DNS zones have zone files that define them.

DNS zone transfers using the AXFR protocol are the simplest mechanism to replicate DNS records across DNS servers. To avoid the need to edit information on multiple DNS servers, you can edit information on one server and use AXFR to copy information to other servers.

**Why is it needed? =>** If a DNS server for a zone is not working and cached information has expired, the domain is inaccessible to all services (web, mail, and more). Therefore, each zone should have at least two DNS servers.

You can use different mechanisms for DNS zone transfer but the simplest one is AXFR (technically speaking, AXFR refers to the protocol used during a DNS zone transfer)

dig +short ns zonetransfer.me\
dig axfr zonetransfer.me @nsztm1.digi.ninja

<div align="left">

<img src="../.gitbook/assets/image (118).png" alt="">

</div>

**AXFR Vulnerability =>** AXFR offers no authentication, so any client can ask a DNS server for a copy of the entire zone. This means that unless some kind of protection is introduced, an attacker can get a list of all hosts for a domain, which gives them a lot of potential attack vectors.

In order to prevent this vulnerability from occurring, the DNS server should be configured to only allow zone transfers from trusted IP addresses.
