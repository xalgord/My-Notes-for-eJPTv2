# HTTP

No need for an explanation about HTTP.

Get the service version of HTTP:

<pre class="language-sh"><code class="lang-sh">root@attackdefense:~# nmap 10.5.18.137 -p80 -sV
Starting Nmap 7.91 ( https://nmap.org ) at 2023-12-28 22:28 IST
Nmap scan report for 10.5.18.137
Host is up (0.0017s latency).

PORT   STATE SERVICE VERSION
<strong>80/tcp open  http    Microsoft IIS httpd 10.0
</strong>Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows
</code></pre>

So it is running Microsoft IIS httpd 10.0

Another tool we can use for further enumeration is whatweb:

```sh
root@attackdefense:~# whatweb 10.5.18.137
Ignoring eventmachine-1.3.0.dev.1 because its extensions are not built. Try: gem pristine eventmachine --version 1.3.0.dev.1
Ignoring fxruby-1.6.29 because its extensions are not built. Try: gem pristine fxruby --version 1.6.29
http://10.5.18.137 [302 Found] ASP_NET[4.0.30319], Cookies[ASP.NET_SessionId,Server], Country[RESERVED][ZZ], HTTPServer[Microsoft-IIS/10.0], HttpOnly[ASP.NET_SessionId], IP[10.5.18.137], Microsoft-IIS[10.0], RedirectLocation[/Default.aspx], Title[Object moved], X-Powered-By[ASP.NET], X-XSS-Protection[0]
http://10.5.18.137/Default.aspx [302 Found] ASP_NET[4.0.30319], Cookies[ASP.NET_SessionId,Server], Country[RESERVED][ZZ], HTTPServer[Microsoft-IIS/10.0], HttpOnly[ASP.NET_SessionId], IP[10.5.18.137], Microsoft-IIS[10.0], RedirectLocation[/Default.aspx], Title[Object moved], X-Powered-By[ASP.NET], X-XSS-Protection[0]
```

Another tool with its usage is here:

```sh
root@attackdefense:~# http 10.5.18.137
HTTP/1.1 302 Found
Cache-Control: private
Content-Length: 130
Content-Type: text/html; charset=utf-8
Date: Thu, 28 Dec 2023 17:08:09 GMT
Location: /Default.aspx
Server: Microsoft-IIS/10.0
Set-Cookie: ASP.NET_SessionId=zyhyvxlmuqaqrfetmjtz24aw; path=/; HttpOnly; SameSite=Lax
Set-Cookie: Server=RE9UTkVUR09BVA==; path=/
X-AspNet-Version: 4.0.30319
X-Powered-By: ASP.NET
X-XSS-Protection: 0

<html><head><title>Object moved</title></head><body>
<h2>Object moved to <a href="/Default.aspx">here</a>.</h2>
</body></html>
```

Another tool on the list is `dirb` that can be utilized for finding directories through brute force.

```sh
root@attackdefense:~# dirb http://10.5.18.137

START_TIME: Thu Dec 28 22:41:38 2023
URL_BASE: http://10.5.18.137/
WORDLIST_FILES: /usr/share/dirb/wordlists/common.txt

-----------------

GENERATED WORDS: 4612                                                          

---- Scanning URL: http://10.5.18.137/ ----
==> DIRECTORY: http://10.5.18.137/app_themes/                                                           
==> DIRECTORY: http://10.5.18.137/aspnet_client/                                                        
==> DIRECTORY: http://10.5.18.137/configuration/                                                        
==> DIRECTORY: http://10.5.18.137/content/                                                              
==> DIRECTORY: http://10.5.18.137/Content/                                                              
==> DIRECTORY: http://10.5.18.137/downloads/                                                            
==> DIRECTORY: http://10.5.18.137/Downloads/                                                            
==> DIRECTORY: http://10.5.18.137/resources/                                                            
==> DIRECTORY: http://10.5.18.137/Resources/                                                            
.
.
```

We can utilize Nmap script to get more info rather than just service info, like common directories etc.

```sh
root@attackdefense:~# nmap 10.5.28.34 -p80 -sV --script=http-enum
Starting Nmap 7.91 ( https://nmap.org ) at 2023-12-28 23:01 IST
Nmap scan report for 10.5.28.34
Host is up (0.0027s latency).

PORT   STATE SERVICE VERSION
80/tcp open  http    Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
| http-enum: 
|   /content/: Potentially interesting folder
|   /downloads/: Potentially interesting folder
|_  /webdav/: Potentially interesting folder
|_http-server-header: Microsoft-IIS/10.0
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows
```

Similar to this, we can use another script for the HTTP Headers:

```sh
root@attackdefense:~# nmap 10.5.28.34 -p80 -sV --script=http-headers
Starting Nmap 7.91 ( https://nmap.org ) at 2023-12-28 23:03 IST
Nmap scan report for 10.5.28.34
Host is up (0.0022s latency).

PORT   STATE SERVICE VERSION
80/tcp open  http    Microsoft IIS httpd 10.0
| http-headers: 
|   Cache-Control: private
|   Content-Type: text/html; charset=utf-8
|   Location: /Default.aspx
|   Server: Microsoft-IIS/10.0
|   Set-Cookie: ASP.NET_SessionId=fyrdvc1dlbswiit3ebbdwmcz; path=/; HttpOnly; SameSite=Lax
|   X-AspNet-Version: 4.0.30319
|   Set-Cookie: Server=RE9UTkVUR09BVA==; path=/
|   X-XSS-Protection: 0
|   X-Powered-By: ASP.NET
|   Date: Thu, 28 Dec 2023 17:34:05 GMT
|   Connection: close
|   Content-Length: 130
|   
|_  (Request type: GET)
|_http-server-header: Microsoft-IIS/10.0
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows
```

Similar to this, we also have a script for HTTP Methods:

```sh
root@attackdefense:~# nmap 10.5.28.34 -p80 -sV --script=http-methods --script-args http-methods.url-path=/webdav/
Starting Nmap 7.91 ( https://nmap.org ) at 2023-12-28 23:08 IST
Nmap scan report for 10.5.28.34
Host is up (0.0023s latency).

PORT   STATE SERVICE VERSION
80/tcp open  http    Microsoft IIS httpd 10.0
| http-methods: 
|   Supported Methods: OPTIONS TRACE GET HEAD POST COPY PROPFIND DELETE MOVE PROPPATCH MKCOL LOCK UNLOCK PUT
|   Potentially risky methods: TRACE COPY PROPFIND DELETE MOVE PROPPATCH MKCOL LOCK UNLOCK PUT
|_  Path tested: /webdav/
|_http-server-header: Microsoft-IIS/10.0
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows
```

To know what methods are allowed in the `webdav`, we can use the following command:

```sh
root@attackdefense:~# nmap 10.5.28.34 -p80 -sV --script=http-webdav-scan --script-args http-methods.url-path=/webdav/
Starting Nmap 7.91 ( https://nmap.org ) at 2023-12-28 23:10 IST
Nmap scan report for 10.5.28.34
Host is up (0.0021s latency).

PORT   STATE SERVICE VERSION
80/tcp open  http    Microsoft IIS httpd 10.0
|_http-server-header: Microsoft-IIS/10.0
| http-webdav-scan: 
|   Public Options: OPTIONS, TRACE, GET, HEAD, POST, PROPFIND, PROPPATCH, MKCOL, PUT, DELETE, COPY, MOVE, LOCK, UNLOCK
|   Server Date: Thu, 28 Dec 2023 17:40:49 GMT
|   WebDAV type: Unknown
|   Allowed Methods: OPTIONS, TRACE, GET, HEAD, POST, COPY, PROPFIND, LOCK, UNLOCK
|_  Server Type: Microsoft-IIS/10.0
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows
```

## HTTP: Apache

```sh
root@INE:~# nmap 192.70.140.3 -sV -p80
Starting Nmap 7.92 ( https://nmap.org ) at 2023-12-28 23:25 IST
Nmap scan report for a4b5s866j2q10xh267zrxh8c1.temp-network_a-70-140 (192.70.140.3)
Host is up (0.000037s latency).

PORT   STATE SERVICE VERSION
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
MAC Address: 02:42:C0:46:8C:03 (Unknown)
```

To get the banner information, we can use the following command:

<pre class="language-sh"><code class="lang-sh">root@INE:~# nmap 192.70.140.3 -sV -p80 --script=banner
Starting Nmap 7.92 ( https://nmap.org ) at 2023-12-28 23:27 IST
Nmap scan report for a4b5s866j2q10xh267zrxh8c1.temp-network_a-70-140 (192.70.140.3)
Host is up (0.000039s latency).

PORT   STATE SERVICE VERSION
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
<strong>|_http-server-header: Apache/2.4.18 (Ubuntu)
</strong>MAC Address: 02:42:C0:46:8C:03 (Unknown)
</code></pre>

Let's try it with Metasploit:

```sh
msf6 > use auxiliary/scanner/http/http_version 
msf6 auxiliary(scanner/http/http_version) > set rhosts 192.70.140.3
rhosts => 192.70.140.3
msf6 auxiliary(scanner/http/http_version) > run

[+] 192.70.140.3:80 Apache/2.4.18 (Ubuntu)
[*] Scanned 1 of 1 hosts (100% complete)
[*] Auxiliary module execution completed
```

So, nothing new, but as expected.

Let's now use curl:

```sh
root@INE:~# curl 192.70.140.3 | more
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100 11321  100 11321    0     0  13.7M      0 --:--:-- --:--:-- --:--:-- 10.7M

<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
<html xmlns="http://www.w3.org/1999/xhtml">
  <!--
    Modified from the Debian original for Ubuntu
    Last updated: 2014-03-19
    See: https://launchpad.net/bugs/1288690
  -->
  <head>
    <meta http-equiv="Content-Type" content="text/html; charset=UTF-8" />
    <title>Apache2 Ubuntu Default Page: It works</title>
    <style type="text/css" media="screen">
  * {
    margin: 0px 0px 0px 0px;
    padding: 0px 0px 0px 0px;
  }

  body, html {
    padding: 3px 3px 3px 3px;

    background-color: #D8DBE2;

    font-family: Verdana, sans-serif;
    font-size: 11pt;
    text-align: center;
  }
.
.
```

Just a simple command. Another simple command to download a webpage:

```sh
root@INE:~# wget "http://192.70.140.3/index"
--2023-12-28 23:35:12--  http://192.70.140.3/index
Connecting to 192.70.140.3:80... connected.
HTTP request sent, awaiting response... 200 OK
Length: 11321 (11K) [text/html]
Saving to: ‘index’

index                                          100%[====================================================================================================>]  11.06K  --.-KB/s    in 0s      

2023-12-28 23:35:12 (234 MB/s) - ‘index’ saved [11321/11321]
```

We can also use Metasploit for the directory brute force just like `dirb`:

```sh
msf6 > use auxiliary/scanner/http/brute_dirs 
msf6 auxiliary(scanner/http/brute_dirs) > set rhosts 192.70.140.3
rhosts => 192.70.140.3
msf6 auxiliary(scanner/http/brute_dirs) > run

[*] Using code '404' as not found.
[+] Found http://192.70.140.3:80/dir/ 200
[+] Found http://192.70.140.3:80/src/ 200
[*] Scanned 1 of 1 hosts (100% complete)
[*] Auxiliary module execution completed
```
