# SSH

> SSH, or Secure Shell Protocol, is a network protocol that allows secure communication between two devices. It is commonly used to remotely access and control servers, allowing users to securely transfer files, execute commands, and manage remote systems. SSH uses encryption to protect sensitive information and prevent unauthorized access, making it a popular choice for secure remote access in the IT industry.

```sh
root@attackdefense:~# nmap 192.41.246.3 -p22 -sV
Starting Nmap 7.70 ( https://nmap.org ) at 2023-12-28 12:57 UTC
Nmap scan report for target-1 (192.41.246.3)
Host is up (0.000060s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.6 (Ubuntu Linux; protocol 2.0)
MAC Address: 02:42:C0:29:F6:03 (Unknown)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 0.51 seconds
```

We have OpenSSH running on port 22 for SSH.

We can use netcat to grab the banner:

```sh
root@attackdefense:~# nc 192.41.246.3 22
SSH-2.0-OpenSSH_7.2p2 Ubuntu-4ubuntu2.6
```

or we can simply use this command to connect to SSH:

```sh
root@attackdefense:~# ssh root@192.41.246.3
```

To enumerate all the algorithms we can use Nmap:

```sh
root@attackdefense:~# nmap 192.41.246.3 -p22 --script=ssh2-enum-algos
Starting Nmap 7.70 ( https://nmap.org ) at 2023-12-28 13:15 UTC
Nmap scan report for target-1 (192.41.246.3)
Host is up (0.000058s latency).

PORT   STATE SERVICE
22/tcp open  ssh
| ssh2-enum-algos: 
|   kex_algorithms: (6)
|       curve25519-sha256@libssh.org
|       ecdh-sha2-nistp256
|       ecdh-sha2-nistp384
|       ecdh-sha2-nistp521
|       diffie-hellman-group-exchange-sha256
|       diffie-hellman-group14-sha1
|   server_host_key_algorithms: (5)
|       ssh-rsa
|       rsa-sha2-512
|       rsa-sha2-256
|       ecdsa-sha2-nistp256
|       ssh-ed25519
|   encryption_algorithms: (6)
|       chacha20-poly1305@openssh.com
|       aes128-ctr
|       aes192-ctr
|       aes256-ctr
|       aes128-gcm@openssh.com
|       aes256-gcm@openssh.com
|   mac_algorithms: (10)
|       umac-64-etm@openssh.com
|       umac-128-etm@openssh.com
|       hmac-sha2-256-etm@openssh.com
|       hmac-sha2-512-etm@openssh.com
|       hmac-sha1-etm@openssh.com
|       umac-64@openssh.com
|       umac-128@openssh.com
|       hmac-sha2-256
|       hmac-sha2-512
|       hmac-sha1
|   compression_algorithms: (2)
|       none
|_      zlib@openssh.com
MAC Address: 02:42:C0:29:F6:03 (Unknown)
```

these are the algorithms that can be used to get that key.

Now to get the RSA key we can use the following command:

```sh
root@attackdefense:~# nmap 192.41.246.3 -p22 --script=ssh-hostkey --script-args ssh_hostkey=full
Starting Nmap 7.70 ( https://nmap.org ) at 2023-12-28 13:20 UTC
Nmap scan report for target-1 (192.41.246.3)
Host is up (0.000064s latency).

PORT   STATE SERVICE
22/tcp open  ssh
| ssh-hostkey: 
|   ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQC1fkJK7F8yxf3vewEcLYHljBnKTAiRqzFxkFo6lqyew73ATL2Abyh6at/oOmBSlPI90rtAMA6jQGJ+0HlHgf7mkjz5+CBo9j2VPu1bejYtcxpqpHcL5Bp12wgey1zup74fgd+yOzILjtgbnDOw1+HSkXqN79d+4BnK0QF6T9YnkHvBhZyjzIDmjonDy92yVBAIoB6Rdp0w7nzFz3aN9gzB5MW/nSmgc4qp7R6xtzGaqZKp1H3W3McZO3RELjGzvHOdRkAKL7n2kyVAraSUrR0Oo5m5e/sXrITYi9y0X6p2PTUfYiYvgkv/3xUF+5YDDA33AJvv8BblnRcRRZ74BxaD
|   ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBB0cJ/kSOXBWVIBA2QH4UB6r7nFL5l7FwHubbSZ9dIs2JSmn/oIgvvQvxmI5YJxkdxRkQlF01KLDmVgESYXyDT4=
|_  ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIKuZlCFfTgeaMC79zla20ZM2q64mjqWhKPw/2UzyQ2W/
MAC Address: 02:42:C0:29:F6:03 (Unknown)
```

We got the full RSA host key.

To check for weak passwords we can utilize the following script:

```sh
root@attackdefense:~# nmap 192.41.246.3 -p22 --script=ssh-auth-methods --script-args="ssh.user=student"
Starting Nmap 7.70 ( https://nmap.org ) at 2023-12-28 13:23 UTC
Nmap scan report for target-1 (192.41.246.3)
Host is up (0.000048s latency).

PORT   STATE SERVICE
22/tcp open  ssh
| ssh-auth-methods: 
|_  Supported authentication methods: none_auth
MAC Address: 02:42:C0:29:F6:03 (Unknown)
```

Here we can see that there is no supported authentication for student which is dangerous.

Let's try it for admin:

```sh
root@attackdefense:~# nmap 192.41.246.3 -p22 --script=ssh-auth-methods --script-args="ssh.user=admin"
Starting Nmap 7.70 ( https://nmap.org ) at 2023-12-28 13:26 UTC
Nmap scan report for target-1 (192.41.246.3)
Host is up (0.000051s latency).

PORT   STATE SERVICE
22/tcp open  ssh
| ssh-auth-methods: 
|   Supported authentication methods: 
|     publickey
|_    password
MAC Address: 02:42:C0:29:F6:03 (Unknown)
```

So here we can see it needs authentication.

Well, we saw that `student` needs no authentication. So let's try to connect with `student`:&#x20;

```sh
root@attackdefense:~# ssh student@192.41.246.3
Welcome to attack defense ssh recon lab!!
Welcome to Ubuntu 16.04.5 LTS (GNU/Linux 5.4.0-153-generic x86_64)
.
.
student@victim-1:~$ whoami
student
```

We are in.
