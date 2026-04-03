# Service Enumeration

Identifying a service is only the first step. What matters more is knowing exactly which version of that service is running. Version information allows you to search for known vulnerabilities specific to that software release, find public exploits that match the target environment, and in some cases locate the source code for that exact version to review it manually. A precise version number gives you a much narrower and more actionable attack surface than a generic service name alone.

---

## Service Version Detection

The recommended approach is to start with a quick port scan to get a general overview of what is open. This generates less traffic and reduces the chance of triggering security monitoring or getting blocked early in the engagement. Once you have that initial picture, you can run a full port scan covering all ports (-p-) combined with version detection (-sV) to extract detailed service and version information across every open port.

Full port scans take a significant amount of time depending on the target. Nmap provides a few ways to monitor progress during long-running scans.

---

### Basic Full Port Version Scan

**Command:**

```bash
sudo nmap 192.168.100.59 -p- -sV
```

**Output:**

```
Starting Nmap 7.98 ( https://nmap.org ) at 2026-04-02 23:01 -0400
Nmap scan report for 192.168.100.59
Host is up (0.0025s latency).
Not shown: 65526 closed tcp ports (reset)
PORT     STATE SERVICE     VERSION
22/tcp   open  ssh         OpenSSH 5.3p1 Debian 3ubuntu4 (Ubuntu Linux; protocol 2.0)
80/tcp   open  http        Apache httpd 2.2.14 ((Ubuntu) mod_mono/2.4.3 PHP/5.3.2-1ubuntu4.30 with Suhosin-Patch proxy_html/3.0.1 mod_python/3.3.1 Python/2.6.5 mod_ssl/2.2.14 OpenSSL...)
139/tcp  open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
143/tcp  open  imap        Courier Imapd (released 2008)
443/tcp  open  ssl/http    Apache httpd 2.2.14 ((Ubuntu) mod_mono/2.4.3 PHP/5.3.2-1ubuntu4.30 with Suhosin-Patch proxy_html/3.0.1 mod_python/3.3.1 Python/2.6.5 mod_ssl/2.2.14 OpenSSL...)
445/tcp  open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
5001/tcp open  java-object Java Object Serialization
8080/tcp open  http        Apache Tomcat/Coyote JSP engine 1.1
8081/tcp open  http        Jetty 6.1.25
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port5001-TCP:V=7.98%I=7%D=4/2%Time=69CF2D78%P=x86_64-pc-linux-gnu%r(NUL
SF:L,4,"\xac\xed\0\x05");
MAC Address: 08:00:27:00:14:D9 (Oracle VirtualBox virtual NIC)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 17.72 seconds

```

---

### Automatic Progress Updates

**Command:**

```bash
sudo nmap 192.168.100.59 -p- -sV --stats-every=5s
```

**Output:**

```
Starting Nmap 7.98 ( https://nmap.org ) at 2026-04-02 23:01 -0400
Stats: 0:00:05 elapsed; 0 hosts completed (1 up), 1 undergoing SYN Stealth Scan
SYN Stealth Scan Timing: About 92.18% done; ETC: 23:01 (0:00:00 remaining)
Stats: 0:00:11 elapsed; 0 hosts completed (1 up), 1 undergoing Service Scan
Service scan Timing: About 33.33% done; ETC: 23:02 (0:00:12 remaining)
Stats: 0:00:16 elapsed; 0 hosts completed (1 up), 1 undergoing Service Scan
Service scan Timing: About 77.78% done; ETC: 23:02 (0:00:03 remaining)
Nmap scan report for 192.168.100.59
Host is up (0.00067s latency).
Not shown: 65526 closed tcp ports (reset)
PORT     STATE SERVICE     VERSION
22/tcp   open  ssh         OpenSSH 5.3p1 Debian 3ubuntu4 (Ubuntu Linux; protocol 2.0)
80/tcp   open  http        Apache httpd 2.2.14 ((Ubuntu) mod_mono/2.4.3 PHP/5.3.2-1ubuntu4.30 with Suhosin-Patch proxy_html/3.0.1 mod_python/3.3.1 Python/2.6.5 mod_ssl/2.2.14 OpenSSL...)
139/tcp  open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
143/tcp  open  imap        Courier Imapd (released 2008)
443/tcp  open  ssl/http    Apache httpd 2.2.14 ((Ubuntu) mod_mono/2.4.3 PHP/5.3.2-1ubuntu4.30 with Suhosin-Patch proxy_html/3.0.1 mod_python/3.3.1 Python/2.6.5 mod_ssl/2.2.14 OpenSSL...)
445/tcp  open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
5001/tcp open  java-object Java Object Serialization
8080/tcp open  http        Apache Tomcat/Coyote JSP engine 1.1
8081/tcp open  http        Jetty 6.1.25
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port5001-TCP:V=7.98%I=7%D=4/2%Time=69CF2DA7%P=x86_64-pc-linux-gnu%r(NUL
SF:L,4,"\xac\xed\0\x05");
MAC Address: 08:00:27:00:14:D9 (Oracle VirtualBox virtual NIC)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 18.14 seconds
                                                               
```

---

### Verbose Mode for Live Port Discovery

**Command:**

```bash
sudo nmap 192.168.100.59 -p- -sV -v
```

**Output:**

```
Starting Nmap 7.98 ( https://nmap.org ) at 2026-04-02 23:02 -0400
NSE: Loaded 48 scripts for scanning.
Initiating ARP Ping Scan at 23:02
Scanning 192.168.100.59 [1 port]
Completed ARP Ping Scan at 23:02, 0.12s elapsed (1 total hosts)
Initiating Parallel DNS resolution of 1 host. at 23:02
Completed Parallel DNS resolution of 1 host. at 23:02, 0.50s elapsed
Initiating SYN Stealth Scan at 23:02
Scanning 192.168.100.59 [65535 ports]
Discovered open port 443/tcp on 192.168.100.59
Discovered open port 22/tcp on 192.168.100.59
Discovered open port 445/tcp on 192.168.100.59
Discovered open port 80/tcp on 192.168.100.59
Discovered open port 139/tcp on 192.168.100.59
Discovered open port 8080/tcp on 192.168.100.59
Discovered open port 143/tcp on 192.168.100.59
Discovered open port 5001/tcp on 192.168.100.59
Discovered open port 8081/tcp on 192.168.100.59
Completed SYN Stealth Scan at 23:02, 3.76s elapsed (65535 total ports)
Initiating Service scan at 23:02
Scanning 9 services on 192.168.100.59
Completed Service scan at 23:02, 12.12s elapsed (9 services on 1 host)
NSE: Script scanning 192.168.100.59.
Initiating NSE at 23:02
Completed NSE at 23:02, 0.22s elapsed
Initiating NSE at 23:02
Completed NSE at 23:02, 0.11s elapsed
Nmap scan report for 192.168.100.59
Host is up (0.00079s latency).
Not shown: 65526 closed tcp ports (reset)
PORT     STATE SERVICE     VERSION
22/tcp   open  ssh         OpenSSH 5.3p1 Debian 3ubuntu4 (Ubuntu Linux; protocol 2.0)
80/tcp   open  http        Apache httpd 2.2.14 ((Ubuntu) mod_mono/2.4.3 PHP/5.3.2-1ubuntu4.30 with Suhosin-Patch proxy_html/3.0.1 mod_python/3.3.1 Python/2.6.5 mod_ssl/2.2.14 OpenSSL...)
139/tcp  open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
143/tcp  open  imap        Courier Imapd (released 2008)
443/tcp  open  ssl/http    Apache httpd 2.2.14 ((Ubuntu) mod_mono/2.4.3 PHP/5.3.2-1ubuntu4.30 with Suhosin-Patch proxy_html/3.0.1 mod_python/3.3.1 Python/2.6.5 mod_ssl/2.2.14 OpenSSL...)
445/tcp  open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
5001/tcp open  java-object Java Object Serialization
8080/tcp open  http        Apache Tomcat/Coyote JSP engine 1.1
8081/tcp open  http        Jetty 6.1.25
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port5001-TCP:V=7.98%I=7%D=4/2%Time=69CF2DD3%P=x86_64-pc-linux-gnu%r(NUL
SF:L,4,"\xac\xed\0\x05");
MAC Address: 08:00:27:00:14:D9 (Oracle VirtualBox virtual NIC)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Read data files from: /usr/share/nmap
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 17.60 seconds
           Raw packets sent: 65536 (2.884MB) | Rcvd: 65536 (2.621MB)

```
