# Host Discovery

## Network Mapping

Network mapping is a technique used to create a visual representation of a computer network's physical and logical components. It helps in identifying all the devices connected to the network, their IP addresses, and how they are connected to each other.

## Ping sweeps

```sh
$ ping -c 5 142.250.193.78
PING 142.250.193.78 (142.250.193.78) 56(84) bytes of data.
64 bytes from 142.250.193.78: icmp_seq=1 ttl=115 time=17.8 ms
64 bytes from 142.250.193.78: icmp_seq=2 ttl=115 time=17.9 ms
64 bytes from 142.250.193.78: icmp_seq=3 ttl=115 time=16.6 ms
64 bytes from 142.250.193.78: icmp_seq=4 ttl=115 time=21.3 ms
64 bytes from 142.250.193.78: icmp_seq=5 ttl=115 time=17.5 ms

--- 142.250.193.78 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 4006ms
rtt min/avg/max/mdev = 16.559/18.209/21.317/1.622 ms
```

<mark style="background-color:yellow;">**Note:**</mark> <mark style="background-color:yellow;"></mark><mark style="background-color:yellow;">Windows might block the ICMP requests.</mark>

To run wireshark with a particular interface, we can use the following command:

```sh
$ sudo wireshark -i eth1
```

To scan the entire subnet, just put the 0 at the end of the IP address, and because this is a broadcast network, we need to specify it using `-b` the argument:

```sh
$ ping -b -c 1 142.250.193.0
```

Another way to do this is by using `fping` tool, which is an improved version of the ping utility.

```sh
$ fping -a -g 142.250.193.0/24
```

`-a` - show targets that are alive

`-g` - generate target lists

If we want a clear view of only alive hosts, then we can use following Linux command:

```
$ fping -a -g 142.250.193.0/24 2>/dev/null
```

## Nmap Host Discovery

```
$ nmap -sn -PS80,3389,445 10.4.23.227
```

or

```
$ nmap -sn -PS1-1000 10.4.23.227
```

`-sn` - To not perform a port scan

`-PS` - TCP SYN Ping
