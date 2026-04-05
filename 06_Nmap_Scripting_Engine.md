# Nmap Scripting Engine

The Nmap Scripting Engine, commonly referred to as NSE, extends Nmap far beyond basic port scanning. It allows users to run prewritten scripts against discovered services to gather additional information, test for vulnerabilities, perform brute force attempts, and much more. These scripts are written in the Lua programming language and are organized into 14 categories based on their purpose and behavior.

Understanding which category to use helps narrow down the right approach for any given situation without running unnecessary or harmful scripts against a target.

| Category | Description |
|----------|-------------|
| auth | Determines authentication credentials for services |
| broadcast | Discovers hosts by broadcasting on the network and adds them to the scan |
| brute | Attempts to log in to services by trying multiple credential combinations |
| default | Scripts that run automatically when using the -sC option |
| discovery | Evaluates and enumerates accessible services |
| dos | Checks services for denial of service vulnerabilities, used cautiously |
| exploit | Attempts to exploit known vulnerabilities on scanned ports |
| external | Uses external third-party services for additional processing |
| fuzzer | Sends unexpected or malformed inputs to identify vulnerabilities |
| intrusive | Scripts that may disrupt or negatively affect the target system |
| malware | Checks whether the target system shows signs of malware infection |
| safe | Non-intrusive scripts that gather information without causing harm |
| version | Extends service detection capabilities |
| vuln | Identifies known specific vulnerabilities on the target |

Scripts can be invoked in three main ways depending on how precise you need to be.

**Run all default scripts:**

```bash
sudo nmap <target> -sC
```

**Run all scripts from a specific category:**

```bash
sudo nmap <target> --script <category>
```

**Run specific named scripts:**

```bash
sudo nmap <target> --script <script-name>,<script-name>,...
```

---

## Default Scripts

Running Nmap with the `-sC` flag activates the default script category, which covers a broad range of common checks. These scripts are selected by the Nmap team as safe and useful enough to run without explicit permission concerns. The results give a solid overview of the services running on the target without being invasive.

**Command:**

```bash
sudo nmap 192.168.100.59 -sC
```

**Output:**

```
Starting Nmap 7.98 ( https://nmap.org ) at 2026-04-05 02:00 -0400
Nmap scan report for 192.168.100.59
Host is up (0.00098s latency).
Not shown: 991 closed tcp ports (reset)
PORT     STATE SERVICE
22/tcp   open  ssh
| ssh-hostkey:
|   1024 ea:83:1e:45:5a:a6:8c:43:1c:3c:e3:18:dd:fc:88:a5 (DSA)
|_  2048 3a:94:d8:3f:e0:a2:7a:b8:c3:94:d7:5e:00:55:0c:a7 (RSA)
80/tcp   open  http
| http-methods:
|_  Potentially risky methods: TRACE
|_http-title: owaspbwa OWASP Broken Web Applications
139/tcp  open  netbios-ssn
143/tcp  open  imap
|_imap-capabilities: THREAD=ORDEREDSUBJECT completed QUOTA ACL OK CAPABILITY THREAD=REFERENCES ACL2=UNIONA0001 CHILDREN IDLE UIDPLUS IMAP4rev1 SORT NAMESPACE
443/tcp  open  https
|_http-title: owaspbwa OWASP Broken Web Applications
| http-methods:
|_  Potentially risky methods: TRACE
| ssl-cert: Subject: commonName=owaspbwa
| Not valid before: 2013-01-02T21:12:38
|_Not valid after:  2022-12-31T21:12:38
|_ssl-date: 2026-04-05T06:00:01+00:00; 0s from scanner time.
445/tcp  open  microsoft-ds
5001/tcp open  commplex-link
8080/tcp open  http-proxy
|_http-title: Site doesn't have a title.
8081/tcp open  blackice-icecap
MAC Address: 08:00:27:00:14:D9 (Oracle VirtualBox virtual NIC)
Host script results:
|_nbstat: NetBIOS name: OWASPBWA, NetBIOS user: <unknown>, NetBIOS MAC: <unknown> (unknown)
|_smb2-time: Protocol negotiation failed (SMB2)
| smb-security-mode:
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
Nmap done: 1 IP address (1 host up) scanned in 30.13 seconds
```

**What this output reveals:**

The scan identifies nine open ports across several services. Port 22 is running SSH with both a 1024-bit DSA key and a 2048-bit RSA key. The presence of a 1024-bit DSA key is worth noting because key sizes of that length are considered weak by modern standards and should be replaced.

Port 80 and 443 are both serving the OWASP Broken Web Applications project, which is an intentionally vulnerable web application environment. The TRACE method is flagged on both ports as potentially risky. The HTTP TRACE method can be abused in certain cross-site scripting attacks known as Cross-Site Tracing, so its presence is a finding worth documenting.

The SSL certificate on port 443 expired at the end of 2022, meaning the HTTPS configuration is significantly out of date and will trigger browser security warnings for any user visiting the site.

Port 445 running SMB has message signing disabled. This is a meaningful security gap because SMB message signing prevents relay attacks where an attacker intercepts authentication traffic and reuses it against another system. With signing disabled, the server is vulnerable to NTLM relay attacks.

The host script results also show the NetBIOS name of the machine as OWASPBWA, and SMB2 protocol negotiation failed, suggesting the system may be running an older SMB implementation that does not support the newer protocol version.

---

## Specifying Scripts by Name

When you know exactly which scripts you want to run, naming them directly gives you focused, clean output without the noise of an entire category. The following scan runs three HTTP-related scripts against the web ports on the target.

**Command:**

```bash
sudo nmap 192.168.100.59 -p 80,8080,8081 --script http-enum,http-methods,http-title
```

**Output:**

```
Starting Nmap 7.98 ( https://nmap.org ) at 2026-04-05 02:08 -0400
Nmap scan report for 192.168.100.59
Host is up (0.00078s latency).
PORT     STATE SERVICE
80/tcp   open  http
| http-methods:
|   Supported Methods: GET HEAD POST OPTIONS TRACE
|_  Potentially risky methods: TRACE
|_http-title: owaspbwa OWASP Broken Web Applications
| http-enum:
|   /wordpress/: Blog
|   /test/: Test page
|   /mono/: Mono
|   /crossdomain.xml: Adobe Flash crossdomain policy
|   /phpmyadmin/: phpMyAdmin
|   /wordpress/wp-login.php: Wordpress login page.
|   /cgi-bin/: Potentially interesting folder w/ directory listing
|   /icons/: Potentially interesting folder w/ directory listing
|_  /images/: Potentially interesting folder w/ directory listing
8080/tcp open  http-proxy
|_http-title: Site doesn't have a title.
| http-enum:
|   /examples/: Sample scripts
|   /manager/html/upload: Apache Tomcat (401 Unauthorized)
|   /manager/html: Apache Tomcat (401 Unauthorized)
|_  /docs/: Potentially interesting folder
8081/tcp open  blackice-icecap
MAC Address: 08:00:27:00:14:D9 (Oracle VirtualBox virtual NIC)
Nmap done: 1 IP address (1 host up) scanned in 34.31 seconds
```

**What this output reveals:**

The `http-enum` script works by requesting a large list of common paths and recording which ones return valid responses. On port 80, it found several significant directories and files. The WordPress installation at `/wordpress/` and its login page at `/wordpress/wp-login.php` are immediately relevant since WordPress login pages are common targets for credential brute force attacks. The `/phpmyadmin/` path is present, which is a database administration interface that should never be publicly accessible on a production or exposed system. The `/cgi-bin/` directory has directory listing enabled, meaning anyone can browse its contents directly without authentication, which is a basic but serious misconfiguration.

On port 8080, the script identified an Apache Tomcat installation. The Tomcat manager interface at `/manager/html` and the upload endpoint at `/manager/html/upload` both returned 401 Unauthorized, meaning they exist but require credentials. The Tomcat manager is a well-known target because default or weak credentials can give an attacker the ability to deploy a malicious web application directly to the server and achieve remote code execution.

The `http-methods` script confirmed that TRACE is enabled on port 80, reinforcing what the default scan found earlier.

---

## SSL Certificate and Cipher Enumeration

For HTTPS services, dedicated SSL scripts provide a much deeper analysis than the default scan. The following command checks the certificate details and enumerates every cipher suite the server supports.

**Command:**

```bash
sudo nmap 192.168.100.59 -p 443 --script ssl-cert,ssl-enum-ciphers,http-title
```

**Output:**

```
Starting Nmap 7.98 ( https://nmap.org ) at 2026-04-05 02:10 -0400
Nmap scan report for 192.168.100.59
Host is up (0.00050s latency).
PORT    STATE SERVICE
443/tcp open  https
| ssl-cert: Subject: commonName=owaspbwa
| Issuer: commonName=owaspbwa
| Public Key type: rsa
| Public Key bits: 1024
| Signature Algorithm: sha1WithRSAEncryption
| Not valid before: 2013-01-02T21:12:38
| Not valid after:  2022-12-31T21:12:38
| MD5:     0fb9 ca0b e9b7 b26f de6c 3555 6186 2399
| SHA-1:   e469 e1f2 9877 40c3 3aec ee7c f630 ca19 31be 05ae
|_SHA-256: b094 5e82 0894 9294 ec14 b1fc d299 8bf1 4833 3ebb 7d34 1351 88e2 98b4 fe2d 46b2
|_http-title: owaspbwa OWASP Broken Web Applications
| ssl-enum-ciphers:
|   SSLv3:
|     ciphers:
|       TLS_DHE_RSA_WITH_3DES_EDE_CBC_SHA (dh 1024) - F
|       TLS_DHE_RSA_WITH_AES_128_CBC_SHA (dh 1024) - F
|       TLS_DHE_RSA_WITH_AES_256_CBC_SHA (dh 1024) - F
|       TLS_RSA_WITH_3DES_EDE_CBC_SHA (rsa 1024) - F
|       TLS_RSA_WITH_AES_128_CBC_SHA (rsa 1024) - F
|       TLS_RSA_WITH_AES_256_CBC_SHA (rsa 1024) - F
|       TLS_RSA_WITH_RC4_128_MD5 (rsa 1024) - F
|       TLS_RSA_WITH_RC4_128_SHA (rsa 1024) - F
|     compressors:
|       DEFLATE
|       NULL
|     cipher preference: client
|     warnings:
|       64-bit block cipher 3DES vulnerable to SWEET32 attack
|       Broken cipher RC4 is deprecated by RFC 7465
|       CBC-mode cipher in SSLv3 (CVE-2014-3566)
|       Ciphersuite uses MD5 for message integrity
|       Insecure certificate signature (SHA1), score capped at F
|   TLSv1.0:
|     ciphers:
|       TLS_DHE_RSA_WITH_3DES_EDE_CBC_SHA (dh 1024) - F
|       TLS_DHE_RSA_WITH_AES_128_CBC_SHA (dh 1024) - F
|       TLS_DHE_RSA_WITH_AES_256_CBC_SHA (dh 1024) - F
|       TLS_RSA_WITH_3DES_EDE_CBC_SHA (rsa 1024) - F
|       TLS_RSA_WITH_AES_128_CBC_SHA (rsa 1024) - F
|       TLS_RSA_WITH_AES_256_CBC_SHA (rsa 1024) - F
|       TLS_RSA_WITH_RC4_128_MD5 (rsa 1024) - F
|       TLS_RSA_WITH_RC4_128_SHA (rsa 1024) - F
|     compressors:
|       DEFLATE
|       NULL
|     cipher preference: client
|     warnings:
|       64-bit block cipher 3DES vulnerable to SWEET32 attack
|       Broken cipher RC4 is deprecated by RFC 7465
|       Ciphersuite uses MD5 for message integrity
|       Insecure certificate signature (SHA1), score capped at F
|_  least strength: F
MAC Address: 08:00:27:00:14:D9 (Oracle VirtualBox virtual NIC)
Nmap done: 1 IP address (1 host up) scanned in 1.43 seconds
```

**What this output reveals:**

The SSL certificate was issued by the server itself, meaning it is self-signed. Self-signed certificates provide no third-party verification of identity. The certificate uses a 1024-bit RSA key, which is no longer considered secure. Modern standards require a minimum of 2048 bits. The certificate also expired at the end of 2022 and has been invalid for years.

The cipher enumeration shows the server still supports SSLv3 and TLSv1.0, both of which are deprecated and insecure. Every cipher suite across both protocol versions received an F grade. The 3DES cipher is vulnerable to the SWEET32 attack, which exploits birthday-bound weaknesses in 64-bit block ciphers. RC4 has been prohibited by RFC 7465 due to well-documented cryptographic weaknesses. MD5 is broken for integrity purposes. SSLv3 with CBC-mode ciphers is directly vulnerable to the POODLE attack documented under CVE-2014-3566. SHA1 as a certificate signature algorithm is also deprecated and no longer trusted by modern browsers.

---

## Aggressive Scan

The `-A` flag enables OS detection, version detection, default script scanning, and traceroute all in one command. It provides broad coverage in a single run at the cost of generating more traffic.

**Command:**

```bash
sudo nmap 192.168.100.59 -p 80 -A
```

**Output:**

```
Starting Nmap 7.98 ( https://nmap.org ) at 2026-04-05 02:12 -0400
Nmap scan report for 192.168.100.59
Host is up (0.00054s latency).
PORT   STATE SERVICE VERSION
80/tcp open  http    Apache httpd 2.2.14 ((Ubuntu) mod_mono/2.4.3 PHP/5.3.2-1ubuntu4.30 with Suhosin-Patch proxy_html/3.0.1 mod_python/3.3.1 Python/2.6.5 mod_ssl/2.2.14 OpenSSL...)
| http-methods:
|_  Potentially risky methods: TRACE
|_http-server-header: Apache/2.2.14 (Ubuntu) mod_mono/2.4.3 PHP/5.3.2-1ubuntu4.30 with Suhosin-Patch proxy_html/3.0.1 mod_python/3.3.1 Python/2.6.5 mod_ssl/2.2.14 OpenSSL/0.9.8k Phusion_Passenger/4.0.38 mod_perl/2.0.4 Perl/v5.10.1
|_http-title: owaspbwa OWASP Broken Web Applications
MAC Address: 08:00:27:00:14:D9 (Oracle VirtualBox virtual NIC)
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose
Running: Linux 2.6.X
OS CPE: cpe:/o:linux:linux_kernel:2.6
OS details: Linux 2.6.17 - 2.6.36
Network Distance: 1 hop
TRACEROUTE
HOP RTT     ADDRESS
1   0.54 ms 192.168.100.59
OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 9.46 seconds
```

**What this output reveals:**

The version string for Apache is extremely detailed. The server is running Apache 2.2.14 on Ubuntu with additional modules including mod_mono, PHP 5.3.2, mod_python, mod_ssl with OpenSSL 0.9.8k, Phusion Passenger, and mod_perl. Every one of these components is significantly outdated. Apache 2.2 reached end of life years ago. PHP 5.3.2 has not received security updates for a very long time. OpenSSL 0.9.8k is an ancient release carrying numerous known vulnerabilities. Exposing all of this version detail in a server header is a security concern on its own because it gives an attacker a complete inventory to research exploits against.

OS detection identified the system as running a Linux 2.6 kernel between versions 2.6.17 and 2.6.36. This is an extremely old kernel, further confirming this system has not received security updates in many years.

The traceroute shows a single hop, confirming the target is on the same local network segment as the scanner.

---

## Vulnerability Assessment

The `vuln` category runs scripts specifically designed to identify known vulnerabilities and map them to CVEs and public exploit databases. This is one of the most directly useful categories during a penetration test because it produces concrete, actionable findings. The following scan targets port 80 against the full version string of the Apache server.

**Command:**

```bash
sudo nmap 192.168.100.59 -p 80 -sV --script vuln
```

**Output:**

```
Starting Nmap 7.98 ( https://nmap.org ) at 2026-04-05 02:15 -0400
Nmap scan report for 192.168.100.59
Host is up (0.00057s latency).

PORT   STATE SERVICE VERSION
80/tcp open  http    Apache httpd 2.2.14 ((Ubuntu) mod_mono/2.4.3 PHP/5.3.2-1ubuntu4.30 with Suhosin-Patch proxy_html/3.0.1 mod_python/3.3.1 Python/2.6.5 mod_ssl/2.2.14 OpenSSL...)
| http-cookie-flags: 
|   /mono/: 
|     ASP.NET_SessionId: 
|_      httponly flag not set
| http-cross-domain-policy: 
|   VULNERABLE:
|   Cross-domain and Client Access policies.
|     State: VULNERABLE
|       A cross-domain policy file specifies the permissions that a web client such as Java, Adobe Flash, Adobe Reader,
|       etc. use to access data across different domains. A client acces policy file is similar to cross-domain policy
|       but is used for M$ Silverlight applications. Overly permissive configurations enables Cross-site Request
|       Forgery attacks, and may allow third parties to access sensitive data meant for the user.
|     Check results:
|       /crossdomain.xml:
|         <?xml version="1.0"?>
|         <!DOCTYPE cross-domain-policy SYSTEM "http://www.macromedia.com/xml/dtds/cross-domain-policy.dtd">
|         <cross-domain-policy>
|           <allow-access-from domain="*" />
|         </cross-domain-policy>
|     Extra information:
|       Trusted domains:*
|   
|     References:
|       https://www.owasp.org/index.php/Test_RIA_cross_domain_policy_%28OTG-CONFIG-008%29
|       https://www.adobe.com/devnet-docs/acrobatetk/tools/AppSec/CrossDomain_PolicyFile_Specification.pdf
|       http://sethsec.blogspot.com/2014/03/exploiting-misconfigured-crossdomainxml.html
|       http://acunetix.com/vulnerabilities/web/insecure-clientaccesspolicy-xml-file
|       http://gursevkalra.blogspot.com/2013/08/bypassing-same-origin-policy-with-flash.html
|_      https://www.adobe.com/devnet/articles/crossdomain_policy_file_spec.html
|_http-trace: TRACE is enabled
|_http-server-header: Apache/2.2.14 (Ubuntu) mod_mono/2.4.3 PHP/5.3.2-1ubuntu4.30 with Suhosin-Patch proxy_html/3.0.1 mod_python/3.3.1 Python/2.6.5 mod_ssl/2.2.14 OpenSSL/0.9.8k Phusion_Passenger/4.0.38 mod_perl/2.0.4 Perl/v5.10.1
| http-csrf: 
| Spidering limited to: maxdepth=3; maxpagecount=20; withinhost=192.168.100.59
|   Found the following possible CSRF vulnerabilities: 
|     
|     Path: http://192.168.100.59:80/shepherd/login.jsp
|     Form id: 
|     Form action: login
|     
|     Path: http://192.168.100.59:80/WackoPicko/
|     Form id: query2
|     Form action: /WackoPicko/pictures/search.php
|     
|     Path: http://192.168.100.59:80/WackoPicko/
|     Form id: 
|     Form action: /WackoPicko/pic' + 'check' + '.php
|     
|     Path: http://192.168.100.59:80/dom-xss-example.html
|     Form id: 
|     Form action: '+location.href+'
|     
|     Path: http://192.168.100.59:80/AppSensorDemo/login.jsp
|     Form id: 
|     Form action: Login
|     
|     Path: http://192.168.100.59:80/wordpress/
|     Form id: searchform
|_    Form action: http://192.168.100.59/wordpress/
| http-internal-ip-disclosure: 
|_  Internal IP Leaked: 127.0.1.1
| vulners: 
|   cpe:/a:apache:http_server:2.2.14: 
|     	SSV:69341	10.0	https://vulners.com/seebug/SSV:69341	*EXPLOIT*
|     	SSV:19282	10.0	https://vulners.com/seebug/SSV:19282	*EXPLOIT*
|     	SSV:19236	10.0	https://vulners.com/seebug/SSV:19236	*EXPLOIT*
|     	PACKETSTORM:86964	10.0	https://vulners.com/packetstorm/PACKETSTORM:86964	*EXPLOIT*
|     	PACKETSTORM:180533	10.0	https://vulners.com/packetstorm/PACKETSTORM:180533	*EXPLOIT*
|     	MSF:AUXILIARY-DOS-HTTP-APACHE_MOD_ISAPI-	10.0	https://vulners.com/metasploit/MSF:AUXILIARY-DOS-HTTP-APACHE_MOD_ISAPI-	*EXPLOIT*
|     	HTTPD:E74B6F3660D13C4DD05DF3A83EA61631	10.0	https://vulners.com/httpd/HTTPD:E74B6F3660D13C4DD05DF3A83EA61631
|     	EXPLOITPACK:30ED468EC8BD5B71B2CB93825A852B80	10.0	https://vulners.com/exploitpack/EXPLOITPACK:30ED468EC8BD5B71B2CB93825A852B80	*EXPLOIT*
|     	EDB-ID:14288	10.0	https://vulners.com/exploitdb/EDB-ID:14288	*EXPLOIT*
|     	EDB-ID:11650	10.0	https://vulners.com/exploitdb/EDB-ID:11650	*EXPLOIT*
|     	CVE-2010-0425	10.0	https://vulners.com/cve/CVE-2010-0425
|     	3E6BA608-776F-5B1F-9BA5-589CD2A5A351	10.0	https://vulners.com/gitee/3E6BA608-776F-5B1F-9BA5-589CD2A5A351	*EXPLOIT*
|     	PACKETSTORM:171631	9.8	https://vulners.com/packetstorm/PACKETSTORM:171631	*EXPLOIT*
|     	HTTPD:E69E9574251973D5AF93FA9D04997FC1	9.8	https://vulners.com/httpd/HTTPD:E69E9574251973D5AF93FA9D04997FC1
|     	HTTPD:E162D3AE025639FEE2A89D5AF40ABF2F	9.8	https://vulners.com/httpd/HTTPD:E162D3AE025639FEE2A89D5AF40ABF2F
|     	HTTPD:C072933AA965A86DA3E2C9172FFC1569	9.8	https://vulners.com/httpd/HTTPD:C072933AA965A86DA3E2C9172FFC1569
|     	HTTPD:A1BBCE110E077FFBF4469D4F06DB9293	9.8	https://vulners.com/httpd/HTTPD:A1BBCE110E077FFBF4469D4F06DB9293
|     	HTTPD:A09F9CEBE0B7C39EDA0480FEAEF4FE9D	9.8	https://vulners.com/httpd/HTTPD:A09F9CEBE0B7C39EDA0480FEAEF4FE9D
|     	HTTPD:9F5406E0F4A0B007A0A4C9C92EF9813B	9.8	https://vulners.com/httpd/HTTPD:9F5406E0F4A0B007A0A4C9C92EF9813B
|     	HTTPD:9BCBE3C14201AFC4B0F36F15CB40C0F8	9.8	https://vulners.com/httpd/HTTPD:9BCBE3C14201AFC4B0F36F15CB40C0F8
|     	HTTPD:2BE0032A6ABE7CC52906DBAAFE0E448E	9.8	https://vulners.com/httpd/HTTPD:2BE0032A6ABE7CC52906DBAAFE0E448E
|     	EDB-ID:51193	9.8	https://vulners.com/exploitdb/EDB-ID:51193	*EXPLOIT*
|     	ECC3E825-EE29-59D3-BE28-1B30DB15940E	9.8	https://vulners.com/githubexploit/ECC3E825-EE29-59D3-BE28-1B30DB15940E	*EXPLOIT*
|     	D5084D51-C8DF-5CBA-BC26-ACF2E33F8E52	9.8	https://vulners.com/githubexploit/D5084D51-C8DF-5CBA-BC26-ACF2E33F8E52	*EXPLOIT*
|     	CVE-2024-38476	9.8	https://vulners.com/cve/CVE-2024-38476
|     	CVE-2022-31813	9.8	https://vulners.com/cve/CVE-2022-31813
|     	CVE-2022-22720	9.8	https://vulners.com/cve/CVE-2022-22720
|     	CVE-2021-44790	9.8	https://vulners.com/cve/CVE-2021-44790
|     	CVE-2021-39275	9.8	https://vulners.com/cve/CVE-2021-39275
|     	CVE-2018-1312	9.8	https://vulners.com/cve/CVE-2018-1312
|     	CVE-2017-7679	9.8	https://vulners.com/cve/CVE-2017-7679
|     	CVE-2017-3169	9.8	https://vulners.com/cve/CVE-2017-3169
|     	CVE-2017-3167	9.8	https://vulners.com/cve/CVE-2017-3167
|     	CNVD-2022-51061	9.8	https://vulners.com/cnvd/CNVD-2022-51061
|     	CNVD-2022-03225	9.8	https://vulners.com/cnvd/CNVD-2022-03225
|     	CNVD-2021-102386	9.8	https://vulners.com/cnvd/CNVD-2021-102386
|     	B6297446-2DDD-52BA-B508-29A748A5D2CC	9.8	https://vulners.com/githubexploit/B6297446-2DDD-52BA-B508-29A748A5D2CC	*EXPLOIT*
|     	1337DAY-ID-38427	9.8	https://vulners.com/zdt/1337DAY-ID-38427	*EXPLOIT*
|     	0DB60346-03B6-5FEE-93D7-FF5757D225AA	9.8	https://vulners.com/gitee/0DB60346-03B6-5FEE-93D7-FF5757D225AA	*EXPLOIT*
|     	HTTPD:509B04B8CC51879DD0A561AC4FDBE0A6	9.1	https://vulners.com/httpd/HTTPD:509B04B8CC51879DD0A561AC4FDBE0A6
|     	HTTPD:459EB8D98503A2460C9445C5B224979E	9.1	https://vulners.com/httpd/HTTPD:459EB8D98503A2460C9445C5B224979E
|     	HTTPD:2C227652EE0B3B961706AAFCACA3D1E1	9.1	https://vulners.com/httpd/HTTPD:2C227652EE0B3B961706AAFCACA3D1E1
|     	FD2EE3A5-BAEA-5845-BA35-E6889992214F	9.1	https://vulners.com/githubexploit/FD2EE3A5-BAEA-5845-BA35-E6889992214F	*EXPLOIT*
|     	FBC8A8BE-F00A-5B6D-832E-F99A72E7A3F7	9.1	https://vulners.com/githubexploit/FBC8A8BE-F00A-5B6D-832E-F99A72E7A3F7	*EXPLOIT*
|     	E606D7F4-5FA2-5907-B30E-367D6FFECD89	9.1	https://vulners.com/githubexploit/E606D7F4-5FA2-5907-B30E-367D6FFECD89	*EXPLOIT*
|     	D8A19443-2A37-5592-8955-F614504AAF45	9.1	https://vulners.com/githubexploit/D8A19443-2A37-5592-8955-F614504AAF45	*EXPLOIT*
|     	CVE-2024-40898	9.1	https://vulners.com/cve/CVE-2024-40898
|     	CVE-2022-28615	9.1	https://vulners.com/cve/CVE-2022-28615
|     	CVE-2022-22721	9.1	https://vulners.com/cve/CVE-2022-22721
|     	CVE-2017-9788	9.1	https://vulners.com/cve/CVE-2017-9788
|     	CNVD-2022-51060	9.1	https://vulners.com/cnvd/CNVD-2022-51060
|     	CNVD-2022-41638	9.1	https://vulners.com/cnvd/CNVD-2022-41638
|     	B5E74010-A082-5ECE-AB37-623A5B33FE7D	9.1	https://vulners.com/githubexploit/B5E74010-A082-5ECE-AB37-623A5B33FE7D	*EXPLOIT*
|     	HTTPD:1B3D546A8500818AAC5B1359FE11A7E4	9.0	https://vulners.com/httpd/HTTPD:1B3D546A8500818AAC5B1359FE11A7E4
|     	FDF3DFA1-ED74-5EE2-BF5C-BA752CA34AE8	9.0	https://vulners.com/githubexploit/FDF3DFA1-ED74-5EE2-BF5C-BA752CA34AE8	*EXPLOIT*
|     	CVE-2021-40438	9.0	https://vulners.com/cve/CVE-2021-40438
|     	CNVD-2022-03224	9.0	https://vulners.com/cnvd/CNVD-2022-03224
|     	AE3EF1CC-A0C3-5CB7-A6EF-4DAAAFA59C8C	9.0	https://vulners.com/githubexploit/AE3EF1CC-A0C3-5CB7-A6EF-4DAAAFA59C8C	*EXPLOIT*
|     	9D9B3F4D-6B5C-5377-BE39-F1C432C9E457	9.0	https://vulners.com/githubexploit/9D9B3F4D-6B5C-5377-BE39-F1C432C9E457	*EXPLOIT*
|     	8AFB43C5-ABD4-52AD-BB19-24D7884FF2A2	9.0	https://vulners.com/githubexploit/8AFB43C5-ABD4-52AD-BB19-24D7884FF2A2	*EXPLOIT*
|     	7F48C6CF-47B2-5AF9-B6FD-1735FB2A95B2	9.0	https://vulners.com/githubexploit/7F48C6CF-47B2-5AF9-B6FD-1735FB2A95B2	*EXPLOIT*
|     	36618CA8-9316-59CA-B748-82F15F407C4F	9.0	https://vulners.com/githubexploit/36618CA8-9316-59CA-B748-82F15F407C4F	*EXPLOIT*
|     	CVE-2025-58098	8.3	https://vulners.com/cve/CVE-2025-58098
|     	CNVD-2021-102387	8.2	https://vulners.com/cnvd/CNVD-2021-102387
|     	B0A9E5E8-7CCC-5984-9922-A89F11D6BF38	8.2	https://vulners.com/githubexploit/B0A9E5E8-7CCC-5984-9922-A89F11D6BF38	*EXPLOIT*
|     	HTTPD:30E0EE442FF4843665FED4FBCA25406A	8.1	https://vulners.com/httpd/HTTPD:30E0EE442FF4843665FED4FBCA25406A
|     	CVE-2016-5387	8.1	https://vulners.com/cve/CVE-2016-5387
|     	CNVD-2016-04948	8.1	https://vulners.com/cnvd/CNVD-2016-04948
|     	SSV:72403	7.8	https://vulners.com/seebug/SSV:72403	*EXPLOIT*
|     	SSV:26043	7.8	https://vulners.com/seebug/SSV:26043	*EXPLOIT*
|     	SSV:20899	7.8	https://vulners.com/seebug/SSV:20899	*EXPLOIT*
|     	PACKETSTORM:180517	7.8	https://vulners.com/packetstorm/PACKETSTORM:180517	*EXPLOIT*
|     	PACKETSTORM:126851	7.8	https://vulners.com/packetstorm/PACKETSTORM:126851	*EXPLOIT*
|     	PACKETSTORM:123527	7.8	https://vulners.com/packetstorm/PACKETSTORM:123527	*EXPLOIT*
|     	PACKETSTORM:122962	7.8	https://vulners.com/packetstorm/PACKETSTORM:122962	*EXPLOIT*
|     	MSF:AUXILIARY-DOS-HTTP-APACHE_RANGE_DOS-	7.8	https://vulners.com/metasploit/MSF:AUXILIARY-DOS-HTTP-APACHE_RANGE_DOS-	*EXPLOIT*
|     	HTTPD:556E7FA885F1BEDB6E3D9AAB5665198F	7.8	https://vulners.com/httpd/HTTPD:556E7FA885F1BEDB6E3D9AAB5665198F
|     	EXPLOITPACK:186B5FCF5C57B52642E62C06BABC6F83	7.8	https://vulners.com/exploitpack/EXPLOITPACK:186B5FCF5C57B52642E62C06BABC6F83	*EXPLOIT*
|     	EDB-ID:18221	7.8	https://vulners.com/exploitdb/EDB-ID:18221	*EXPLOIT*
|     	CVE-2011-3192	7.8	https://vulners.com/cve/CVE-2011-3192
|     	C76F17FD-A21F-5E67-97D8-51A53B9594C1	7.8	https://vulners.com/githubexploit/C76F17FD-A21F-5E67-97D8-51A53B9594C1	*EXPLOIT*
|     	952369B3-F757-55D6-B0C6-9F72C04294A3	7.8	https://vulners.com/githubexploit/952369B3-F757-55D6-B0C6-9F72C04294A3	*EXPLOIT*
|     	4F94F3CE-6A41-5E04-A51B-4D22ED6CF210	7.8	https://vulners.com/githubexploit/4F94F3CE-6A41-5E04-A51B-4D22ED6CF210	*EXPLOIT*
|     	1337DAY-ID-21170	7.8	https://vulners.com/zdt/1337DAY-ID-21170	*EXPLOIT*
|     	SSV:12673	7.5	https://vulners.com/seebug/SSV:12673	*EXPLOIT*
|     	PACKETSTORM:181038	7.5	https://vulners.com/packetstorm/PACKETSTORM:181038	*EXPLOIT*
|     	MSF:AUXILIARY-SCANNER-HTTP-APACHE_OPTIONSBLEED-	7.5	https://vulners.com/metasploit/MSF:AUXILIARY-SCANNER-HTTP-APACHE_OPTIONSBLEED-	*EXPLOIT*
|     	HTTPD:F1CFBC9B54DFAD0499179863D36830BB	7.5	https://vulners.com/httpd/HTTPD:F1CFBC9B54DFAD0499179863D36830BB
|     	EDB-ID:42745	7.5	https://vulners.com/exploitdb/EDB-ID:42745	*EXPLOIT*
|     	CVE-2023-31122	7.5	https://vulners.com/cve/CVE-2023-31122
|     	CVE-2022-30556	7.5	https://vulners.com/cve/CVE-2022-30556
|     	CVE-2022-29404	7.5	https://vulners.com/cve/CVE-2022-29404
|     	CVE-2022-22719	7.5	https://vulners.com/cve/CVE-2022-22719
|     	CVE-2021-34798	7.5	https://vulners.com/cve/CVE-2021-34798
|     	CVE-2018-8011	7.5	https://vulners.com/cve/CVE-2018-8011
|     	CVE-2018-1303	7.5	https://vulners.com/cve/CVE-2018-1303
|     	CVE-2017-9798	7.5	https://vulners.com/cve/CVE-2017-9798
|     	CVE-2017-15710	7.5	https://vulners.com/cve/CVE-2017-15710
|     	CVE-2016-8743	7.5	https://vulners.com/cve/CVE-2016-8743
|     	CVE-2006-20001	7.5	https://vulners.com/cve/CVE-2006-20001
|     	CNVD-2025-30836	7.5	https://vulners.com/cnvd/CNVD-2025-30836
|     	CVE-2025-49812	7.4	https://vulners.com/cve/CVE-2025-49812
|     	CVE-2023-38709	7.3	https://vulners.com/cve/CVE-2023-38709
|     	SSV:60427	6.9	https://vulners.com/seebug/SSV:60427	*EXPLOIT*
|     	CVE-2012-0883	6.9	https://vulners.com/cve/CVE-2012-0883
|     	SSV:12447	6.8	https://vulners.com/seebug/SSV:12447	*EXPLOIT*
|     	CVE-2014-0226	6.8	https://vulners.com/cve/CVE-2014-0226
|     	HTTPD:3E4CF20C0CAD918E98C98926264946F2	6.1	https://vulners.com/httpd/HTTPD:3E4CF20C0CAD918E98C98926264946F2
|     	CVE-2016-4975	6.1	https://vulners.com/cve/CVE-2016-4975
|     	CVE-2018-1302	5.9	https://vulners.com/cve/CVE-2018-1302
|     	CVE-2018-1301	5.9	https://vulners.com/cve/CVE-2018-1301
|     	VULNERLAB:967	5.8	https://vulners.com/vulnerlab/VULNERLAB:967	*EXPLOIT*
|     	CVE-2009-3555	5.8	https://vulners.com/cve/CVE-2009-3555
|     	CNVD-2025-30835	5.4	https://vulners.com/cnvd/CNVD-2025-30835
|     	CVE-2022-37436	5.3	https://vulners.com/cve/CVE-2022-37436
|     	CVE-2022-28614	5.3	https://vulners.com/cve/CVE-2022-28614
|     	CVE-2013-1862	5.1	https://vulners.com/cve/CVE-2013-1862
|     	CVE-2015-3183	5.0	https://vulners.com/cve/CVE-2015-3183
|     	CVE-2014-0231	5.0	https://vulners.com/cve/CVE-2014-0231
|     	CVE-2014-0098	5.0	https://vulners.com/cve/CVE-2014-0098
|     	CVE-2013-6438	5.0	https://vulners.com/cve/CVE-2013-6438
|     	CVE-2013-5704	5.0	https://vulners.com/cve/CVE-2013-5704
|     	CVE-2012-4557	5.0	https://vulners.com/cve/CVE-2012-4557
|     	CVE-2011-3368	5.0	https://vulners.com/cve/CVE-2011-3368
|     	CVE-2010-2068	5.0	https://vulners.com/cve/CVE-2010-2068
|     	CVE-2010-1452	5.0	https://vulners.com/cve/CVE-2010-1452
|     	CVE-2010-0408	5.0	https://vulners.com/cve/CVE-2010-0408
|     	CVE-2009-3720	5.0	https://vulners.com/cve/CVE-2009-3720
|     	CVE-2007-6750	5.0	https://vulners.com/cve/CVE-2007-6750
|     	CVE-2012-0031	4.6	https://vulners.com/cve/CVE-2012-0031
|     	CVE-2011-3607	4.4	https://vulners.com/cve/CVE-2011-3607
|     	CVE-2016-8612	4.3	https://vulners.com/cve/CVE-2016-8612
|     	CVE-2014-0118	4.3	https://vulners.com/cve/CVE-2014-0118
|     	CVE-2013-1896	4.3	https://vulners.com/cve/CVE-2013-1896
|     	CVE-2012-4558	4.3	https://vulners.com/cve/CVE-2012-4558
|     	CVE-2012-3499	4.3	https://vulners.com/cve/CVE-2012-3499
|     	CVE-2012-0053	4.3	https://vulners.com/cve/CVE-2012-0053
|     	CVE-2011-4317	4.3	https://vulners.com/cve/CVE-2011-4317
|     	CVE-2011-3639	4.3	https://vulners.com/cve/CVE-2011-3639
|     	CVE-2011-3348	4.3	https://vulners.com/cve/CVE-2011-3348
|     	CVE-2011-0419	4.3	https://vulners.com/cve/CVE-2011-0419
|     	CVE-2010-0434	4.3	https://vulners.com/cve/CVE-2010-0434
|     	CVE-2008-0455	4.3	https://vulners.com/cve/CVE-2008-0455
|     	CVE-2012-2687	2.6	https://vulners.com/cve/CVE-2012-2687
|     	SSV:60250	1.2	https://vulners.com/seebug/SSV:60250	*EXPLOIT*
|     	CVE-2011-4415	1.2	https://vulners.com/cve/CVE-2011-4415
|     	1337DAY-ID-9602	0.0	https://vulners.com/zdt/1337DAY-ID-9602	*EXPLOIT*
|     	1337DAY-ID-21346	0.0	https://vulners.com/zdt/1337DAY-ID-21346	*EXPLOIT*
|     	1337DAY-ID-17257	0.0	https://vulners.com/zdt/1337DAY-ID-17257	*EXPLOIT*
|     	1337DAY-ID-16843	0.0	https://vulners.com/zdt/1337DAY-ID-16843	*EXPLOIT*
|     	1337DAY-ID-13268	0.0	https://vulners.com/zdt/1337DAY-ID-13268	*EXPLOIT*
|_    	1337DAY-ID-11185	0.0	https://vulners.com/zdt/1337DAY-ID-11185	*EXPLOIT*
|_http-stored-xss: Couldn't find any stored XSS vulnerabilities.
| http-vuln-cve2011-3192: 
|   VULNERABLE:
|   Apache byterange filter DoS
|     State: VULNERABLE
|     IDs:  CVE:CVE-2011-3192  BID:49303
|       The Apache web server is vulnerable to a denial of service attack when numerous
|       overlapping byte ranges are requested.
|     Disclosure date: 2011-08-19
|     References:
|       https://www.securityfocus.com/bid/49303
|       https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2011-3192
|       https://seclists.org/fulldisclosure/2011/Aug/175
|_      https://www.tenable.com/plugins/nessus/55976
| http-enum: 
|   /wordpress/: Blog
|   /test/: Test page
|   /mono/: Mono
|   /crossdomain.xml: Adobe Flash crossdomain policy
|   /phpmyadmin/: phpMyAdmin
|   /wordpress/wp-login.php: Wordpress login page.
|   /cgi-bin/: Potentially interesting folder w/ directory listing
|   /icons/: Potentially interesting folder w/ directory listing
|_  /images/: Potentially interesting folder w/ directory listing
| http-fileupload-exploiter: 
|   
|     Failed to upload and execute a payload.
|   
|     Failed to upload and execute a payload.
|   
|     Failed to upload and execute a payload.
|   
|     Failed to upload and execute a payload.
|   
|     Failed to upload and execute a payload.
|   
|     Failed to upload and execute a payload.
|   
|     Failed to upload and execute a payload.
|   
|     Failed to upload and execute a payload.
|   
|     Failed to upload and execute a payload.
|   
|     Failed to upload and execute a payload.
|   
|     Failed to upload and execute a payload.
|   
|     Failed to upload and execute a payload.
|   
|     Failed to upload and execute a payload.
|   
|     Failed to upload and execute a payload.
|   
|     Failed to upload and execute a payload.
|   
|_    Failed to upload and execute a payload.
| http-sql-injection: 
|   Possible sqli for queries:
|     http://192.168.100.59:80/ESAPI-Java-SwingSet-Interactive/main?function=Introduction%27%20OR%20sqlspider
|     http://192.168.100.59:80/ESAPI-Java-SwingSet-Interactive/main?function=SessionManagement%27%20OR%20sqlspider
|     http://192.168.100.59:80/ESAPI-Java-SwingSet-Interactive/main?function=InitialSetup%27%20OR%20sqlspider
|     http://192.168.100.59:80/ESAPI-Java-SwingSet-Interactive/main?function=Encoding%27%20OR%20sqlspider
|     http://192.168.100.59:80/ESAPI-Java-SwingSet-Interactive/main?function=ValidateUserInput%27%20OR%20sqlspider
|     http://192.168.100.59:80/ESAPI-Java-SwingSet-Interactive/main?function=About%27%20OR%20sqlspider
|     http://192.168.100.59:80/ESAPI-Java-SwingSet-Interactive/main?function=AccessControl%27%20OR%20sqlspider
|     http://192.168.100.59:80/ESAPI-Java-SwingSet-Interactive/main?function=Logging%27%20OR%20sqlspider
|     http://192.168.100.59:80/ESAPI-Java-SwingSet-Interactive/main?function=HttpSecurity%27%20OR%20sqlspider
|     http://192.168.100.59:80/ESAPI-Java-SwingSet-Interactive/main?function=DataProtection%27%20OR%20sqlspider
|     http://192.168.100.59:80/ESAPI-Java-SwingSet-Interactive/main?function=Login%27%20OR%20sqlspider
|_    http://192.168.100.59:80/ESAPI-Java-SwingSet-Interactive/main?function=Encryption%27%20OR%20sqlspider
| http-dombased-xss: 
| Spidering limited to: maxdepth=3; maxpagecount=20; withinhost=192.168.100.59
|   Found the following indications of potential DOM based XSS: 
|     
|     Source: document.write("Hello, " + document.URL.substring(pos,document.URL.length)
|     Pages: http://192.168.100.59:80/dom-xss-example.html
|     
|     Source: document.write('<FORM METHOD="GET" ACTION="'+location.href+'">Enter your name:<input name="name"><input type="submit" value="Submit"></form>')
|_    Pages: http://192.168.100.59:80/dom-xss-example.html
MAC Address: 08:00:27:00:14:D9 (Oracle VirtualBox virtual NIC)

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 58.24 seconds
```

**Option Breakdown:**

| Option | Description |
|--------|-------------|
| 192.168.100.59 | The target being scanned |
| -p 80 | Scans only port 80 |
| -sV | Performs service and version detection |
| --script vuln | Runs all scripts in the vuln category |

**Detailed Explanation of Every Finding:**

**HTTPOnly Flag Not Set on Session Cookie**

The ASP.NET session cookie on the `/mono/` path does not carry the HTTPOnly attribute. This flag instructs the browser to block JavaScript from reading the cookie. Without it, any cross-site scripting vulnerability present anywhere on the application can be used to steal the session token directly through a script, allowing an attacker to take over an authenticated user's session without ever knowing their password.

**Cross-Domain Policy Misconfiguration (VULNERABLE)**

The file at `/crossdomain.xml` contains the directive `<allow-access-from domain="*" />`. This wildcard instructs browser plugins such as Adobe Flash, Adobe Reader, and Microsoft Silverlight to allow any external website to make requests to this server and read the responses. The impact of this misconfiguration is that third-party websites can silently issue requests on behalf of a logged-in user and receive the results, enabling cross-site request forgery and potentially exposing private data that should only be accessible to the authenticated user. A properly configured policy should restrict access to specific trusted domains rather than allowing everyone.

**HTTP TRACE Method Enabled**

The `http-trace` result confirms that the TRACE method is active on the server. TRACE is a diagnostic HTTP method that echoes back the entire request including all headers. The danger is that it can be combined with a cross-site scripting attack in a technique known as Cross-Site Tracing. An attacker can trick a victim's browser into sending a TRACE request which reflects the victim's cookies and authentication headers back through the attacker's script. This effectively bypasses the HttpOnly protection on cookies because the TRACE response body contains the cookie value even when JavaScript cannot access it directly.

**Cross-Site Request Forgery (CSRF)**

The `http-csrf` script spidered up to three levels deep across the web server and identified six separate locations containing forms that are potentially vulnerable to CSRF. The affected paths include login forms on the Shepherd application at `/shepherd/login.jsp`, the AppSensorDemo application at `/AppSensorDemo/login.jsp`, multiple forms within the WackoPicko application, the `dom-xss-example.html` page, and the WordPress installation at `/wordpress/`. A CSRF attack works by tricking a victim who is already authenticated into clicking a crafted link or loading a page that submits one of these forms automatically. Because the browser includes session cookies with every request, the server processes the action as if the victim willingly submitted it. The impact ranges from unauthorized account actions to data modification depending on what each form controls.

**Internal IP Address Disclosure**

The server leaked the internal loopback alias `127.0.1.1` through one of its HTTP response headers. While this particular address is not directly sensitive on its own, internal IP disclosure as a class of issue indicates that the server is not properly stripping or sanitizing headers before returning them to clients. In other environments, this same behavior could leak real private network addresses, revealing the internal network topology and potentially helping an attacker pivot to systems that are not directly accessible from the outside.

**Apache Byterange Filter Denial of Service (CVE-2011-3192) - CONFIRMED VULNERABLE**

This is a confirmed vulnerability with a clear Nmap determination of the VULNERABLE state. The Apache byterange filter in version 2.2.14 fails to enforce limits on the number of overlapping byte ranges a client can request in a single HTTP request. By sending a crafted Range header that specifies hundreds of overlapping segments, an attacker can cause the server to allocate massive amounts of memory to satisfy the request, consuming CPU and memory until the service becomes unresponsive. This vulnerability was publicly disclosed in August 2011 and carries CVE-2011-3192 with a severity score of 7.8. A ready-to-use Metasploit auxiliary module exists for this issue, and multiple packet storm exploits are listed in the vulners output, making it trivially exploitable by anyone with basic tooling.

**File Upload Exploitation Attempts**

The `http-fileupload-exploiter` script identified file upload forms across the web application and attempted to upload and execute a payload through each one. All sixteen attempts failed, which means the script could not achieve code execution through the file upload functionality at this time. However, this does not necessarily mean the upload functionality is fully secure. The script uses a specific approach, and different techniques or file types may still succeed. The finding should be noted and the upload endpoints should be manually tested during the assessment.

**SQL Injection**

The `http-sql-injection` script appended a single quote character to the function parameter across twelve different pages within the ESAPI Java SwingSet Interactive application and observed responses indicating that the input was being interpreted by the database rather than treated as a literal string. The affected URLs include parameters controlling page routing such as Introduction, SessionManagement, InitialSetup, Encoding, ValidateUserInput, About, AccessControl, Logging, HttpSecurity, DataProtection, Login, and Encryption. SQL injection in parameters that control application flow can allow an attacker to extract database contents, bypass authentication, modify stored data, and in some configurations run operating system commands through the database engine.

**DOM-Based Cross-Site Scripting**

The `http-dombased-xss` script found two sources of DOM-based XSS on the page at `/dom-xss-example.html`. The first source uses `document.write` to insert a portion of the current URL directly into the page without any sanitization. The second source uses `document.write` to construct an entire HTML form where the action attribute is populated directly from `location.href`. In both cases, an attacker can craft a URL containing HTML or JavaScript payload characters. When a victim visits that crafted URL, the malicious content is injected into the page by the browser itself, executing in the context of the web application. This can be used to steal cookies, redirect users, or perform actions on the victim's behalf.

**Stored XSS**

The `http-stored-xss` script reported that no stored cross-site scripting vulnerabilities were found during its automated spider. Stored XSS is generally more severe than reflected or DOM-based variants because the malicious script is saved on the server and executes for every user who views the affected page. While the automated check came back clean, manual testing of input fields, comments, profiles, and any other data that gets saved and redisplayed is still recommended since automated tools often miss context-specific injection points.

**Vulners CVE Database Matches**

The vulners script matched the installed Apache 2.2.14 version against its vulnerability database and returned an extensive list of known CVEs, exploit pack entries, Seebug entries, Packet Storm exploits, Metasploit modules, and exploit database entries. The findings span severity scores from 0.0 up to 10.0. The most critical entries are explained below organized by severity tier.

At the maximum score of 10.0, CVE-2010-0425 affects a module bundled with this Apache version and has multiple public exploits available including entries in ExploitDB, Packet Storm, and Seebug. Several Metasploit auxiliary modules also target Apache at this score level, including one for mod_isapi. These are the highest priority issues on the list.

At a score of 9.8, the list includes CVE-2024-38476 which is a recent vulnerability meaning this server version is still receiving new CVE assignments in current years, not just historical ones. CVE-2022-31813 relates to HTTP request smuggling which can be used to bypass security controls and poison shared caches. CVE-2022-22720 is another request smuggling variant. CVE-2021-44790 is a buffer overflow in the mod_lua module. CVE-2021-39275 is an out-of-bounds write. CVE-2018-1312 relates to authentication bypass under certain configurations. CVE-2017-7679 is a buffer overflow in mod_mime that allows remote code execution. CVE-2017-3167 and CVE-2017-3169 both relate to authentication bypass through crafted HTTP requests.

At a score of 9.1, CVE-2024-40898 is another recent finding. CVE-2022-28615 and CVE-2022-22721 both relate to out-of-bounds reads and writes. CVE-2017-9788 is an uninitialized memory flaw in mod_auth_digest that can be triggered remotely.

At a score of 9.0, CVE-2021-40438 is a server-side request forgery vulnerability in mod_proxy. This is particularly significant because SSRF can allow an attacker to reach internal services that are not exposed externally, effectively using the server as a proxy to access backend infrastructure.

At a score of 8.1, CVE-2016-5387 is the HTTPoxy vulnerability affecting CGI applications running under Apache. It allows an attacker to set a proxy through an HTTP header, redirecting backend HTTP requests from scripts to an attacker-controlled server.

At a score of 7.8, CVE-2011-3192 appears again, confirming it in the vulners list alongside its Metasploit module and multiple Packet Storm entries.

At a score of 7.5, multiple CVEs appear covering a broad range of issues including CVE-2023-31122 from 2023, CVE-2017-9798 known as the Optionsbleed vulnerability that leaks memory through the OPTIONS method, and CVE-2016-8743 relating to HTTP request validation bypass.

At lower scores ranging from 4.3 down, a large number of cross-site scripting, information disclosure, and denial of service CVEs are listed. Notable among these is CVE-2012-0053, which leaks httpOnly cookies through error pages under certain conditions, and CVE-2009-3555 at 5.8 which relates to a TLS renegotiation flaw that was a significant issue in its time.

The entries marked with an asterisk and the label EXPLOIT indicate that a working exploit is publicly available for that specific vulnerability. The presence of Metasploit modules in the list is particularly significant from an operational standpoint, as it means these vulnerabilities can be weaponized with minimal technical effort using standard penetration testing tooling.
