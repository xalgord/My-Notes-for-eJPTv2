# Passive

* Use the `host` command to retrieve the IP address of a website, which may include multiple addresses if it's behind a firewall like Cloudflare.

For example:

```sh
➜  ~ host hackersploit.org
hackersploit.org has address 172.67.202.99
hackersploit.org has address 104.21.44.180
hackersploit.org has IPv6 address 2606:4700:3031::6815:2cb4
hackersploit.org has IPv6 address 2606:4700:3036::ac43:ca63
hackersploit.org mail is handled by 0 _dc-mx.2c2a3526b376.hackersploit.org.
```

***

* A helpful tip to begin with is to check files such as **robots.txt** and **sitemap.xml**. These files can provide valuable information about the website's structure and content.
* Firefox addons like BuiltWith and Wappalyzer or command-line tool like WhatWeb can be used to detect web technologies.

```sh
➜  ~ whatweb hackersploit.org
http://hackersploit.org [301 Moved Permanently] Country[UNITED STATES][US], HTTPServer[cloudflare], IP[104.21.44.180], RedirectLocation[https://hackersploit.org/], UncommonHeaders[report-to,nel,cf-ray,alt-svc]
https://hackersploit.org/ [403 Forbidden] Country[UNITED STATES][US], HTML5, HTTPServer[cloudflare], IP[104.21.44.180], Title[403 Forbidden][Title element contains newline(s)!], UncommonHeaders[referrer-policy,x-turbo-charged-by,cf-cache-status,report-to,nel,cf-ray,alt-svc]
```

* We can utilize the whois lookup tool to gather further information about the target.

## Netcraft

Netcraft is a web-based tool that is widely used for footprinting and identifying the technology behind a website. It provides detailed information on web technologies, DNS records, and server information. Netcraft is especially useful for identifying the hosting provider of a given website, as well as the operating system and web server software used.

{% embed url="https://sitereport.netcraft.com/" %}

***

## DNS Recon

DNSRecon is a powerful DNS enumeration tool that is available in Kali Linux. This tool is used to gather information about a target domain by performing different types of DNS queries. DNSRecon can be used to find subdomains, gather information about DNS servers, zone transfers, and much more. It is a command-line tool that requires some technical knowledge of DNS and network protocols. DNSRecon can be a useful tool for penetration testers and security researchers to identify potential weaknesses in a target's DNS infrastructure.

Example:

```sh
➜  ~ dnsrecon -d hackersploit.org
[*] std: Performing General Enumeration against: hackersploit.org...
[*] DNSSEC is configured for hackersploit.org
[*] DNSKEYs:
[*]     None ZSK ECDSAP256SHA256 a09311112cf9138818cd2feae970ebbd 4d6a30f6088c25b325a39abbc5cd1197 aa098283e5aaf421177c2aa5d714992a 9957d1bcc18f98cd71f1f1806b65e148
[*]     None KSk ECDSAP256SHA256 99db2cc14cabdc33d6d77da63a2f15f7 1112584f234e8d1dc428e39e8a4a97e1 aa271a555dc90701e17e2a4c4b6f120b 7c32d44f4ac02bd894cf2d4be7778a19
[*]      SOA dee.ns.cloudflare.com 108.162.192.93
[*]      SOA dee.ns.cloudflare.com 172.64.32.93
[*]      SOA dee.ns.cloudflare.com 173.245.58.93
[*]      SOA dee.ns.cloudflare.com 2a06:98c1:50::ac40:205d
[*]      SOA dee.ns.cloudflare.com 2606:4700:50::adf5:3a5d
[*]      SOA dee.ns.cloudflare.com 2803:f800:50::6ca2:c05d
[*]      NS dee.ns.cloudflare.com 108.162.192.93
[*]      Bind Version for 108.162.192.93 "2023.11.2"
[*]      NS dee.ns.cloudflare.com 172.64.32.93
[*]      Bind Version for 172.64.32.93 "2023.11.2"
[*]      NS dee.ns.cloudflare.com 173.245.58.93
[*]      Bind Version for 173.245.58.93 "2023.11.2"
[*]      NS dee.ns.cloudflare.com 2a06:98c1:50::ac40:205d
[*]      NS dee.ns.cloudflare.com 2606:4700:50::adf5:3a5d
[*]      NS dee.ns.cloudflare.com 2803:f800:50::6ca2:c05d
[*]      NS jim.ns.cloudflare.com 172.64.33.125
[*]      Bind Version for 172.64.33.125 "2023.11.2"
[*]      NS jim.ns.cloudflare.com 173.245.59.125
[*]      Bind Version for 173.245.59.125 "2023.11.2"
[*]      NS jim.ns.cloudflare.com 108.162.193.125
[*]      Bind Version for 108.162.193.125 "2023.11.2"
[*]      NS jim.ns.cloudflare.com 2606:4700:58::adf5:3b7d
[*]      NS jim.ns.cloudflare.com 2803:f800:50::6ca2:c17d
[*]      NS jim.ns.cloudflare.com 2a06:98c1:50::ac40:217d
[*]      MX _dc-mx.2c2a3526b376.hackersploit.org 198.54.120.212
[*]      A hackersploit.org 172.67.202.99
[*]      A hackersploit.org 104.21.44.180
[*]      AAAA hackersploit.org 2606:4700:3036::ac43:ca63
[*]      AAAA hackersploit.org 2606:4700:3031::6815:2cb4
[*]      TXT hackersploit.org google-site-verification=TW0pQsFZ0xx3w4b7kysBV0UrcMq7fJFB-5Rz9h6GwkU
[*]      TXT hackersploit.org v=spf1 +ip4:198.54.120.203 +include:spf.web-hosting.com +ip4:198.54.120.212 +include:hackersploit.org ~all
[*] Enumerating SRV Records
[-] No SRV Records Found for hackersploit.org
```

{% hint style="info" %}
**Important Note**: Cloudflare does not hide MX records by default. MX records, responsible for email routing, may expose the origin server's IP as you can see above.
{% endhint %}

## DNS Dumpster

DNS Dumpster is a web-based tool for DNS enumeration that allows users to search for DNS records related to a specific domain. This tool can be used to find subdomains, MX records, TXT records, and more. DNS Dumpster is different from DNSRecon in that it is a web-based tool that does not require the user to have technical knowledge of DNS and network protocols. Additionally, DNS Dumpster provides a graphical user interface (GUI) that makes it easy to use and understand. DNSRecon, on the other hand, is a command-line tool that requires some technical knowledge of DNS and network protocols to use effectively.

{% embed url="https://dnsdumpster.com/" %}

***

## WAF detection using wafw00f

Usage:

```sh
➜  ~ wafw00f hackersploit.org

[*] Checking https://hackersploit.org
[+] The site https://hackersploit.org is behind Cloudflare (Cloudflare Inc.) WAF.
[~] Number of requests: 2
```

***

## Google Hacking Database

{% embed url="https://www.exploit-db.com/google-hacking-database" %}
