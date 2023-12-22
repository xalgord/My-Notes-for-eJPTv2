# Passive

* Use the `host` command to retrieve the IP address of a website, which may include multiple addresses if it's behind a firewall like Cloudflare.

For example:

```
➜  ~ host hackersploit.org
hackersploit.org has address 172.67.202.99
hackersploit.org has address 104.21.44.180
hackersploit.org has IPv6 address 2606:4700:3031::6815:2cb4
hackersploit.org has IPv6 address 2606:4700:3036::ac43:ca63
hackersploit.org mail is handled by 0 _dc-mx.2c2a3526b376.hackersploit.org.
```

***

* A helpful tip to begin with is to check files such as **robots.txt** and **sitemap.xml**. These files can provide valuable information about the website's structure and content.
* Firefox addons like BuiltWith and Wappalyzer or command-line tool like WhatWeb can be used to detect web technologies.

```
➜  ~ whatweb hackersploit.org
http://hackersploit.org [301 Moved Permanently] Country[UNITED STATES][US], HTTPServer[cloudflare], IP[104.21.44.180], RedirectLocation[https://hackersploit.org/], UncommonHeaders[report-to,nel,cf-ray,alt-svc]
https://hackersploit.org/ [403 Forbidden] Country[UNITED STATES][US], HTML5, HTTPServer[cloudflare], IP[104.21.44.180], Title[403 Forbidden][Title element contains newline(s)!], UncommonHeaders[referrer-policy,x-turbo-charged-by,cf-cache-status,report-to,nel,cf-ray,alt-svc]
```
