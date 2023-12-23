# Active

## DNS Records

* A - Resolves a hostname or domain to an IPv3 address.
* AAAA - Resolves a hostname or domain to an IPv6 address.
* NS - Reference to the domain nameserver.
* MX - Resolves a domain to a mail server.
* CNAME - Used for domain aliases.
* TXT - Text record.
* HINFO - Host information.
* SOA - Domain authority.
* SRV - Service records.
* PTR - Resolves an IP address to a hostname.

## DNS Zone Transfer using dnsenum tool

dnsenum is a tool that is used for DNS zone enumeration and information gathering. It can perform various types of DNS queries to gather information about a domain, such as DNS records, subdomains, and IP addresses. One of its features is the ability to perform a DNS zone transfer, which is a way to obtain a copy of the entire DNS zone file from a DNS server. This can be useful for reconnaissance and vulnerability assessment purposes.

## DNS Zone Transfer using dig

Reference:

{% embed url="https://digi.ninja/projects/zonetransferme.php" %}

***

## Host Discovery with Nmap:

To check your IP address with subnet:

```sh
➜  ~ ip a s
```

To discover active hosts with Nmap:

```sh
➜  ~ nmap -sn 172.21.xxx.x/24
```

`sn` - No port scan

Copy and save the result to perform further port scanning.\


We can also perform the same operation using a different tool called netdiscover:

```sh
~ netdiscover -i eth0 -r 172.21.xxx.x/24
```

## Port Scanning with Nmap

Default Nmap Scan:

```sh
~ nmap -Pn 172.21.xxx.x
```

Use `-Pn` to avoid being blocked by windows. This will skip the check if the target is live or not. This is the default port scan which means it will scan only 1000 most common ports in the network.

To scan all the 65,365 ports, we can use the following command:

```sh
~ nmap -Pn -p- 172.21.xxx.x
```

To scan for specific ports, we can use the following command:

```sh
~ nmap -Pn -p 80,443,3389 172.21.xxx.x
```

To perform a UDP scan, we can use the following command:

```sh
~ nmap -Pn -sU 172.21.xxx.x
```

To identify the specific service versions of ports, we can use the following command:

```sh
~ nmap -Pn -sV 172.21.xxx.x
```

To enable the operating system detection scan, we can use the following command:

```sh
~ nmap -Pn -sV -O 172.21.xxx.x
```

To enable the default nmap script scan, we can use the following command:

```sh
~ nmap -Pn -sV -O -sC 172.21.xxx.x
```

The above scan is also known as the Aggressive scan; instead, we can use the following command:

```sh
~ nmap -Pn -A 172.21.xxx.x
```

To use the Timing template to adjust the speed(T1 to T5), we can use the following command:

```sh
~ nmap -Pn -sV -O -sC -T3 172.21.xxx.x
```

To save the output result to a txt file, we can use the following command:

```sh
~ nmap -Pn -sV -O -sC 172.21.xxx.x -oN test.txt
```

```sh
~ nmap -Pn -sV -O -sC 172.21.xxx.x -oX test.xml
```
