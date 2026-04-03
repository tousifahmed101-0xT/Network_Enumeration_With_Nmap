# Service Enumeration

It is essential to determine the application and its version as accurately as possible. We can use this information to scan for known vulnerabilities and analyze the source code for that version if we find it. An exact version number allows us to search for a more precise exploit that fits the service and the operating system of our target.

## Service Version Detection

It is recommended to perform a quick port scan first, which gives us a small overview of the available ports. This causes significantly less traffic, which is advantageous for us because otherwise we can be discovered and blocked by the security mechanisms. We can deal with these first and then run a full port scan covering all ports (-p-) combined with version detection (-sV) to extract detailed service and version information across every open port.

A full port scan takes quite a long time. To view the scan status, we can press the [Space Bar] during the scan, which will cause Nmap to show us the scan status.

```bash
sudo nmap 192.168.100.59 -p- -sV
```

```
Starting Nmap 7.98 ( https://nmap.org ) at 2026-04-02 23:01 -0400
[Space Bar]
Stats: 0:00:05 elapsed; 0 hosts completed (1 up), 1 undergoing SYN Stealth Scan
SYN Stealth Scan Timing: About 92.18% done; ETC: 23:01 (0:00:00 remaining)
```

| Scanning Options | Description |
|------------------|-------------|
| 192.168.100.59 | Scans the specified target. |
| -p- | Scans all ports. |
| -sV | Performs service version detection on specified ports. |

Another option (`--stats-every=5s`) that we can use is defining how often the status should be shown. Here we can specify the number of seconds (s) or minutes (m) after which we want to get the status.

```bash
sudo nmap 192.168.100.59 -p- -sV --stats-every=5s
```

```
Starting Nmap 7.98 ( https://nmap.org ) at 2026-04-02 23:01 -0400
Stats: 0:00:05 elapsed; 0 hosts completed (1 up), 1 undergoing SYN Stealth Scan
SYN Stealth Scan Timing: About 92.18% done; ETC: 23:01 (0:00:00 remaining)
Stats: 0:00:11 elapsed; 0 hosts completed (1 up), 1 undergoing Service Scan
Service scan Timing: About 33.33% done; ETC: 23:02 (0:00:12 remaining)
Stats: 0:00:16 elapsed; 0 hosts completed (1 up), 1 undergoing Service Scan
Service scan Timing: About 77.78% done; ETC: 23:02 (0:00:03 remaining)
```

| Scanning Options | Description |
|------------------|-------------|
| 192.168.100.59 | Scans the specified target. |
| -p- | Scans all ports. |
| -sV | Performs service version detection on specified ports. |
| --stats-every=5s | Shows the progress of the scan every 5 seconds. |

We can also increase the verbosity level (-v / -vv), which will show us the open ports directly when Nmap detects them.

```bash
sudo nmap 192.168.100.59 -p- -sV -v
```

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
<SNIP>
```

| Scanning Options | Description |
|------------------|-------------|
| 192.168.100.59 | Scans the specified target. |
| -p- | Scans all ports. |
| -sV | Performs service version detection on specified ports. |
| -v | Increases the verbosity of the scan, which displays more detailed information. |

## Banner Grabbing

Once the scan is complete, we will see all TCP ports with the corresponding service and their versions that are active on the system.

```bash
sudo nmap 192.168.100.59 -p- -sV
```

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

| Scanning Options | Description |
|------------------|-------------|
| 192.168.100.59 | Scans the specified target. |
| -p- | Scans all ports. |
| -sV | Performs service version detection on specified ports. |

Primarily, Nmap looks at the banners of the scanned ports and prints them out. If it cannot identify versions through the banners, Nmap attempts to identify them through a signature-based matching system, but this significantly increases the scan duration. One disadvantage to Nmap's presented results is that the automatic scan can miss some information because sometimes Nmap does not know how to handle it. Let us look at an example of this.

```bash
sudo nmap 192.168.100.59 -p- -sV -Pn -n --disable-arp-ping --packet-trace
```

```
Starting Nmap 7.98 ( https://nmap.org ) at 2026-04-02 23:01 -0400
<SNIP>
NSOCK INFO [0.7570s] nsock_iod_new2(): nsock_iod_new (IOD #1)
NSOCK INFO [0.7590s] nsock_connect_tcp(): TCP connection requested to 192.168.100.59:445 (IOD #1) (timeout: 5000ms) EID 8
NSOCK INFO [0.7590s] nsock_trace_handler_callback(): Callback: CONNECT SUCCESS for EID 8 [192.168.100.59:445]
Service scan sending probe NULL to 192.168.100.59:445 (tcp)
NSOCK INFO [0.7590s] nsock_read(): Read request from IOD #1 [192.168.100.59:445] (timeout: 6000ms) EID 18
NSOCK INFO [6.7640s] nsock_trace_handler_callback(): Callback: READ TIMEOUT for EID 18 [192.168.100.59:445]
Service scan sending probe SMBProgNeg to 192.168.100.59:445 (tcp)
NSOCK INFO [6.7640s] nsock_write(): Write request for 168 bytes to IOD #1 EID 27 [192.168.100.59:445]
NSOCK INFO [6.7640s] nsock_read(): Read request from IOD #1 [192.168.100.59:445] (timeout: 5000ms) EID 34
NSOCK INFO [6.7640s] nsock_trace_handler_callback(): Callback: WRITE SUCCESS for EID 27 [192.168.100.59:445]
NSOCK INFO [6.7670s] nsock_trace_handler_callback(): Callback: READ SUCCESS for EID 34 [192.168.100.59:445] (101 bytes)
Service scan hard match (Probe SMBProgNeg matched with SMBProgNeg line 14107): 192.168.100.59:445 is netbios-ssn.  Version: |Samba smbd|3.X - 4.X|workgroup: WORKGROUP|
NSOCK INFO [6.7670s] nsock_iod_delete(): nsock_iod_delete (IOD #1)
Nmap scan report for 192.168.100.59
Host is up, received user-set (0.00057s latency).

PORT    STATE SERVICE     REASON         VERSION
445/tcp open  netbios-ssn syn-ack ttl 64 Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
MAC Address: 40:00:40:06:F0:E1 (Unknown)

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 6.88 seconds
```

| Scanning Options | Description |
|------------------|-------------|
| 192.168.100.59 | Scans the specified target. |
| -p- | Scans all ports. |
| -sV | Performs service version detection on specified ports. |
| -Pn | Disables ICMP Echo requests. |
| -n | Disables DNS resolution. |
| --disable-arp-ping | Disables ARP ping. |
| --packet-trace | Shows all packets sent and received. |

If we look at the results from Nmap, we can see the port's status, service name, and version. Nevertheless, let us look at what the packet trace tells us. After connecting to port 445, Nmap first sent a NULL probe and waited up to 6 seconds for a response. When nothing came back (READ TIMEOUT), it moved on to a more targeted probe called SMBProgNeg, which is a known SMB negotiation packet. The target replied with 101 bytes of data, and Nmap matched that against its signature database to identify the service as Samba smbd 3.X to 4.X.

It happens because after a successful three-way handshake, the server often sends a banner for identification. This serves to let the client know which service it is working with. At the network level, this happens with a PSH flag in the TCP header. However, it can happen that some services do not immediately provide such information. It is also possible to remove or manipulate the banners from the respective services. If we manually connect to a service using nc and grab the banner while intercepting the traffic using tcpdump, we can sometimes see more information than what Nmap showed us.
