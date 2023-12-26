# Port Scanning with Nmap

for the host discovery, use the following command ( if no target ip address is given ):

<pre class="language-sh"><code class="lang-sh">root@attackdefense:~# ip a s
1: lo: &#x3C;LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
2: ip_vti0@NONE: &#x3C;NOARP> mtu 1480 qdisc noop state DOWN group default qlen 1000
    link/ipip 0.0.0.0 brd 0.0.0.0
87644: eth0@if87645: &#x3C;BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default 
    link/ether 02:42:0a:01:00:05 brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 10.1.0.5/16 brd 10.1.255.255 scope global eth0
       valid_lft forever preferred_lft forever
87647: eth1@if87648: &#x3C;BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default 
    link/ether 02:42:c0:6d:f7:02 brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet <a data-footnote-ref href="#user-content-fn-1">192.109.247.2/24</a> brd 192.109.247.255 scope global eth1
       valid_lft forever preferred_lft forever
</code></pre>

Now scan the subnet for active hosts:

<pre class="language-sh"><code class="lang-sh">root@attackdefense:~# nmap -sn 192.109.247.2/24
Starting Nmap 7.70 ( https://nmap.org ) at 2023-12-26 10:27 UTC
Nmap scan report for linux (192.109.247.1)
Host is up (0.000061s latency).
MAC Address: 02:42:9D:34:1B:F0 (Unknown)
Nmap scan report for target-1 (<a data-footnote-ref href="#user-content-fn-2">192.109.247.3</a>)
Host is up (0.000015s latency).
MAC Address: 02:42:C0:6D:F7:03 (Unknown)
Nmap scan report for attackdefense.com (192.109.247.2)
Host is up.
Nmap done: 256 IP addresses (3 hosts up) scanned in 2.02 seconds
</code></pre>

We got our target address i.e.: `192.109.247.3`

Now, we need to scan the entire TCP ports:

```sh
root@attackdefense:~# nmap -sS -T4 -p- 192.109.247.3
Starting Nmap 7.70 ( https://nmap.org ) at 2023-12-26 10:30 UTC
Nmap scan report for target-1 (192.109.247.3)
Host is up (0.0000090s latency).
Not shown: 65532 closed ports
PORT      STATE SERVICE
6421/tcp  open  nim-wan
41288/tcp open  unknown
55413/tcp open  unknown
MAC Address: 02:42:C0:6D:F7:03 (Unknown)

Nmap done: 1 IP address (1 host up) scanned in 1.17 seconds
```

Here, we can see we are getting `unkown` services on the ports, so we need to enable service version detection feature of Nmap:

```sh
root@attackdefense:~# nmap -sS -sV -T4 -p- 192.109.247.3
Starting Nmap 7.70 ( https://nmap.org ) at 2023-12-26 10:32 UTC
Nmap scan report for target-1 (192.109.247.3)
Host is up (0.0000090s latency).
Not shown: 65532 closed ports
PORT      STATE SERVICE   VERSION
6421/tcp  open  mongodb   MongoDB 2.6.10
41288/tcp open  memcached Memcached
55413/tcp open  ftp       vsftpd 3.0.3
MAC Address: 02:42:C0:6D:F7:03 (Unknown)
Service Info: OS: Unix

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 12.44 seconds
```

Now we can see the accurate result.

Here we can also see the OS it is running on. To get the detailed info about OS we can run the following scan:

```sh
root@attackdefense:~# nmap -sS -sV -O -T4 -p- 192.109.247.3
Starting Nmap 7.70 ( https://nmap.org ) at 2023-12-26 10:36 UTC
Nmap scan report for target-1 (192.109.247.3)
Host is up (0.000066s latency).
Not shown: 65532 closed ports
PORT      STATE SERVICE VERSION
6421/tcp  open  mongodb MongoDB 2.6.10
41288/tcp open  achat   AChat chat system
55413/tcp open  ftp     vsftpd 3.0.3
MAC Address: 02:42:C0:6D:F7:03 (Unknown)
No exact OS matches for host (If you know what OS is running on it, see https://nmap.org/submit/ ).
TCP/IP fingerprint:
OS:SCAN(V=7.70%E=4%D=12/26%OT=6421%CT=1%CU=33656%PV=N%DS=1%DC=D%G=Y%M=0242C
OS:0%TM=658AACB3%P=x86_64-pc-linux-gnu)SEQ(SP=103%GCD=1%ISR=10C%TI=Z%CI=Z%I
OS:I=I%TS=A)OPS(O1=M5B4ST11NW7%O2=M5B4ST11NW7%O3=M5B4NNT11NW7%O4=M5B4ST11NW
OS:7%O5=M5B4ST11NW7%O6=M5B4ST11)WIN(W1=FE88%W2=FE88%W3=FE88%W4=FE88%W5=FE88
OS:%W6=FE88)ECN(R=Y%DF=Y%T=40%W=FAF0%O=M5B4NNSNW7%CC=Y%Q=)T1(R=Y%DF=Y%T=40%
OS:S=O%A=S+%F=AS%RD=0%Q=)T2(R=N)T3(R=N)T4(R=Y%DF=Y%T=40%W=0%S=A%A=Z%F=R%O=%
OS:RD=0%Q=)T5(R=Y%DF=Y%T=40%W=0%S=Z%A=S+%F=AR%O=%RD=0%Q=)T6(R=Y%DF=Y%T=40%W
OS:=0%S=A%A=Z%F=R%O=%RD=0%Q=)T7(R=Y%DF=Y%T=40%W=0%S=Z%A=S+%F=AR%O=%RD=0%Q=)
OS:U1(R=Y%DF=N%T=40%IPL=164%UN=0%RIPL=G%RID=G%RIPCK=G%RUCK=G%RUD=G)IE(R=Y%D
OS:FI=N%T=40%CD=S)

Network Distance: 1 hop
Service Info: OS: Unix

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 24.12 seconds
```

It is still unable to accurately tell what OS is running, so we can further use a command to detect the OS aggressively:

```sh
root@attackdefense:~# nmap -sS -sV -O --osscan-guess -T4 -p- 192.109.247.3
Starting Nmap 7.70 ( https://nmap.org ) at 2023-12-26 10:39 UTC
Nmap scan report for target-1 (192.109.247.3)
Host is up (0.000051s latency).
Not shown: 65532 closed ports
PORT      STATE SERVICE   VERSION
6421/tcp  open  mongodb   MongoDB 2.6.10
41288/tcp open  memcached Memcached
55413/tcp open  ftp       vsftpd 3.0.3
MAC Address: 02:42:C0:6D:F7:03 (Unknown)
Aggressive OS guesses: Linux 2.6.32 (96%), Linux 3.2 - 4.9 (96%), Linux 2.6.32 - 3.10 (96%), Linux 3.4 - 3.10 (95%), Linux 3.1 (95%), Linux 3.2 (95%), AXIS 210A or 211 Network Camera (Linux 2.6.17) (94%), Synology DiskStation Manager 5.2-5644 (94%), Netgear RAIDiator 4.2.28 (94%), Linux 2.6.32 - 2.6.35 (94%)
No exact OS matches for host (If you know what OS is running on it, see https://nmap.org/submit/ ).
TCP/IP fingerprint:
OS:SCAN(V=7.70%E=4%D=12/26%OT=6421%CT=1%CU=44124%PV=N%DS=1%DC=D%G=Y%M=0242C
OS:0%TM=658AAD5C%P=x86_64-pc-linux-gnu)SEQ(SP=108%GCD=1%ISR=109%TI=Z%CI=Z%I
OS:I=I%TS=A)OPS(O1=M5B4ST11NW7%O2=M5B4ST11NW7%O3=M5B4NNT11NW7%O4=M5B4ST11NW
OS:7%O5=M5B4ST11NW7%O6=M5B4ST11)WIN(W1=FE88%W2=FE88%W3=FE88%W4=FE88%W5=FE88
OS:%W6=FE88)ECN(R=Y%DF=Y%T=40%W=FAF0%O=M5B4NNSNW7%CC=Y%Q=)T1(R=Y%DF=Y%T=40%
OS:S=O%A=S+%F=AS%RD=0%Q=)T2(R=N)T3(R=N)T4(R=Y%DF=Y%T=40%W=0%S=A%A=Z%F=R%O=%
OS:RD=0%Q=)T5(R=Y%DF=Y%T=40%W=0%S=Z%A=S+%F=AR%O=%RD=0%Q=)T6(R=Y%DF=Y%T=40%W
OS:=0%S=A%A=Z%F=R%O=%RD=0%Q=)T7(R=Y%DF=Y%T=40%W=0%S=Z%A=S+%F=AR%O=%RD=0%Q=)
OS:U1(R=Y%DF=N%T=40%IPL=164%UN=0%RIPL=G%RID=G%RIPCK=G%RUCK=G%RUD=G)IE(R=Y%D
OS:FI=N%T=40%CD=S)

Network Distance: 1 hop
Service Info: OS: Unix

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 24.21 seconds
```

Now we can see, it is trying to guess the version of OS aggressively.

We can also set the intensity for service version detection in Nmap using `--version-detection` the option, where we can set the intensity level between 0-9. The higher the intensity level increases the possibility of correctness.

```sh
root@attackdefense:~# nmap -sS -sV --version-intensity 8 -O --osscan-guess -T4 -p- 192.109.247.3
```

## Nmap Scripting Engine

> Nmap Scripting Engine is a powerful feature that allows users to write and execute scripts to automate various network discovery and security tasks. The scripting engine provides a flexible and extensible framework that can be used to customize and extend the functionality of Nmap. With the scripting engine, users can create custom scripts to detect vulnerabilities, exploit weaknesses, and gather information about network hosts and services. The scripting engine also allows users to integrate Nmap with other tools and technologies, making it a valuable asset in any security professional's toolkit.

We can find the nmap scripts in the following directory:

```sh
root@attackdefense:~# ls -al /usr/share/nmap/scripts/
```

We haven't got the accurate result for the operating system previously, so we will use script engine to do that job for us:

<pre class="language-sh"><code class="lang-sh">root@attackdefense:~# nmap -sS -sV -sC -T4 -p- 192.109.247.3
Starting Nmap 7.70 ( https://nmap.org ) at 2023-12-26 10:58 UTC
Nmap scan report for target-1 (192.109.247.3)
Host is up (0.0000090s latency).
Not shown: 65532 closed ports
PORT      STATE SERVICE VERSION
6421/tcp  open  mongodb MongoDB 2.6.10 2.6.10
| mongodb-databases: 
|   totalSize = 83886080.0
|   databases
|     1
|       empty = true
|       name = admin
|       sizeOnDisk = 1.0
|     0
|       empty = false
|       name = local
|       sizeOnDisk = 83886080.0
|_  ok = 1.0
| mongodb-info: 
|   MongoDB Build info
|     ok = 1.0
<strong>|     sysInfo = Linux lgw01-12 3.19.0-25-generic #26~14.04.1-Ubuntu SMP Fri Jul 24 21:16:20 UTC 2015 x86_64 BOOST_LIB_VERSION=1_58
</strong>|     loaderFlags = -fPIC -pthread -Wl,-z,now -rdynamic
|     versionArray
|       1 = 6
|       2 = 10
|       3 = 0
|       0 = 2
|     gitVersion = nogitversion
|     OpenSSLVersion = OpenSSL 1.0.2g  1 Mar 2016
|     version = 2.6.10
|     javascriptEngine = V8
|     maxBsonObjectSize = 16777216
|     allocator = tcmalloc
    .
    .
    .
</code></pre>

Now we got the accurate result for the target OS name and its version which is `14.04.1-Ubuntu`.

[^1]: 

[^2]: target address
