# Firewall/IDS Evasion

Starting from the quick host discovery using nmap:

```sh
root@attackdefense:~# nmap -sn 10.5.27.2 
Starting Nmap 7.91 ( https://nmap.org ) at 2023-12-26 16:54 IST
Nmap scan report for 10.5.27.2
Host is up (0.0023s latency).
Nmap done: 1 IP address (1 host up) scanned in 0.12 seconds
```

We can see the host is up.

We can use these techniques in Nmap if we find any filtered port.

```
FIREWALL/IDS EVASION AND SPOOFING:
  -f; --mtu <val>: fragment packets (optionally w/given MTU)
  -D <decoy1,decoy2[,ME],...>: Cloak a scan with decoys
  -S <IP_Address>: Spoof source address
  -e <iface>: Use specified interface
  -g/--source-port <portnum>: Use given port number
  --proxies <url1,[url2],...>: Relay connections through HTTP/SOCKS4 proxies
  --data <hex string>: Append a custom payload to sent packets
  --data-string <string>: Append a custom ASCII string to sent packets
  --data-length <num>: Append random data to sent packets
  --ip-options <options>: Send packets with specified ip options
  --ttl <val>: Set IP time-to-live field
  --spoof-mac <mac address/prefix/vendor name>: Spoof your MAC address
  --badsum: Send packets with a bogus TCP/UDP/SCTP checksum
```

For example, we can use the following command:

```sh
root@attackdefense:~# nmap -Pn -sS -sV -p445,3389 -f --data-length 200 -g 53 -D 10.10.12.1,10.10.12.2 10.5.27.2
Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times will be slower.
Starting Nmap 7.91 ( https://nmap.org ) at 2023-12-26 17:15 IST
Nmap scan report for 10.5.27.2
Host is up (0.0023s latency).

PORT     STATE SERVICE            VERSION
445/tcp  open  microsoft-ds       Microsoft Windows Server 2008 R2 - 2012 microsoft-ds
3389/tcp open  ssl/ms-wbt-server?
Service Info: OS: Windows Server 2008 R2 - 2012; CPE: cpe:/o:microsoft:windows

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 65.35 seconds
```

* `-Pn`: This option disables host discovery and marks all addresses as "up." This makes the scan slower but more thorough.
* `-sS`: This option specifies the type of scan to perform, in this case, a TCP SYN scan.
* `-sV`: This option enables version detection, which attempts to identify the versions of the services running on the target ports.
* `-p445,3389`: This option specifies the ports to scan, in this case, ports 445 and 3389.
* `-f`: This option enables the fragmentation of packets sent during the scan. This can help bypass certain types of firewall rules.
* `--data-length 200`: This option specifies the length of data to send in each packet. This can help detect certain types of firewall filtering.
* `-g 53`: This option specifies a source port to use for the scan, in this case, port 53. This can help bypass certain types of firewall rules.
* `-D 10.10.12.1,10.10.12.2`: This option specifies a list of decoy IP addresses to use during the scan, in this case, `10.10.12.1` and `10.10.12.2`. This can help hide the true source of the scan and make it more difficult to detect.
* `10.5.27.2`: This is the target IP address to scan.

We can get the gateway IP here:

<pre class="language-sh"><code class="lang-sh">root@attackdefense:~# ifconfig
eth0: flags=4163&#x3C;UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 10.1.0.6  netmask 255.255.0.0  broadcast 10.1.255.255
        ether 02:42:0a:01:00:06  txqueuelen 0  (Ethernet)
        RX packets 16991  bytes 1301887 (1.2 MiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 17184  bytes 6020512 (5.7 MiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

eth1: flags=4163&#x3C;UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet <a data-footnote-ref href="#user-content-fn-1">10.10.12.3</a>  netmask 255.255.255.0  broadcast 10.10.12.255
        ether 02:42:0a:0a:0c:03  txqueuelen 0  (Ethernet)
        RX packets 21  bytes 1406 (1.3 KiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 6  bytes 292 (292.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

lo: flags=73&#x3C;UP,LOOPBACK,RUNNING>  mtu 65536
        inet 127.0.0.1  netmask 255.0.0.0
        loop  txqueuelen 1000  (Local Loopback)
        RX packets 69045  bytes 160842203 (153.3 MiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 69045  bytes 160842203 (153.3 MiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

</code></pre>

[^1]: This is the gateway IP
