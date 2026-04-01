# Service Enumeration

Identifying a service is only the first step. What matters more is knowing exactly which version of that service is running. Version information allows you to search for known vulnerabilities specific to that software release, find public exploits that match the target environment, and in some cases locate the source code for that exact version to review it manually. A precise version number gives you a much narrower and more actionable attack surface than a generic service name alone.

---

## Service Version Detection

The recommended approach is to start with a quick port scan to get a general overview of what is open. This generates less traffic and reduces the chance of triggering security monitoring or getting blocked early in the engagement. Once you have that initial picture, you can run a full port scan in the background covering all ports (-p-) while working with what you already know. The version scan (-sV) is then used against specific ports to extract detailed service and version information.

Full port scans take a significant amount of time depending on the target. Nmap provides a few ways to monitor progress during long-running scans.

---

### Basic Full Port Version Scan

**Command:**

```bash
sudo nmap 172.16.5.20 -p- -sV
```

**Output (mid-scan after pressing Space Bar):**

```
Starting Nmap 7.80 ( https://nmap.org ) at 2020-06-15 19:44 CEST
Stats: 0:00:03 elapsed; 0 hosts completed (1 up), 1 undergoing SYN Stealth Scan
SYN Stealth Scan Timing: About 3.64% done; ETC: 19:45 (0:00:53 remaining)
```

**Option Breakdown:**

| Option | Description |
|--------|-------------|
| 172.16.5.20 | The target being scanned |
| -p- | Scans all 65535 ports |
| -sV | Performs service and version detection on discovered open ports |

Pressing the Space Bar at any point during an active scan causes Nmap to print the current progress, how far through the scan it is, and the estimated time to completion. This is useful for long scans where you want to check in without interrupting anything.

---

### Automatic Progress Updates

Instead of pressing Space Bar manually, you can tell Nmap to print progress at regular intervals using `--stats-every`. You define the interval in seconds (s) or minutes (m).

**Command:**

```bash
sudo nmap 172.16.5.20 -p- -sV --stats-every=5s
```

**Output:**

```
Starting Nmap 7.80 ( https://nmap.org ) at 2020-06-15 19:46 CEST
Stats: 0:00:05 elapsed; 0 hosts completed (1 up), 1 undergoing SYN Stealth Scan
SYN Stealth Scan Timing: About 13.91% done; ETC: 19:49 (0:00:31 remaining)
Stats: 0:00:10 elapsed; 0 hosts completed (1 up), 1 undergoing SYN Stealth Scan
SYN Stealth Scan Timing: About 39.57% done; ETC: 19:48 (0:00:15 remaining)
```

**Option Breakdown:**

| Option | Description |
|--------|-------------|
| 172.16.5.20 | The target being scanned |
| -p- | Scans all 65535 ports |
| -sV | Performs service and version detection |
| --stats-every=5s | Prints scan progress automatically every 5 seconds |

This is more practical than manually pressing Space Bar during long scans. Setting it to 5 seconds gives frequent updates without cluttering the output too much. For very long scans you might prefer 30s or 1m.

---

### Verbose Mode for Live Port Discovery

Adding verbosity with `-v` causes Nmap to report open ports as soon as they are discovered rather than waiting until the entire scan is finished. This is particularly useful when you want to start investigating individual services while the full scan is still running in the background.

**Command:**

```bash
sudo nmap 172.16.5.20 -p- -sV -v
```

**Output:**

```
Starting Nmap 7.80 ( https://nmap.org ) at 2020-06-15 20:03 CEST
NSE: Loaded 45 scripts for scanning.
Initiating ARP Ping Scan at 20:03
Scanning 172.16.5.20 [1 port]
Completed ARP Ping Scan at 20:03, 0.03s elapsed (1 total hosts)
Initiating Parallel DNS resolution of 1 host. at 20:03
Completed Parallel DNS resolution of 1 host. at 20:03, 0.02s elapsed
Initiating SYN Stealth Scan at 20:03
Scanning 172.16.5.20 [65535 ports]
Discovered open port 995/tcp on 172.16.5.20
Discovered open port 80/tcp on 172.16.5.20
Discovered open port 993/tcp on 172.16.5.20
Discovered open port 143/tcp on 172.16.5.20
Discovered open port 25/tcp on 172.16.5.20
Discovered open port 110/tcp on 172.16.5.20
Discovered open port 22/tcp on 172.16.5.20
<SNIP>
```

**Option Breakdown:**

| Option | Description |
|--------|-------------|
| 172.16.5.20 | The target being scanned |
| -p- | Scans all 65535 ports |
| -sV | Performs service and version detection |
| -v | Increases verbosity, reporting open ports as they are found |

The verbose output also shows each scan phase as it begins, including the ARP ping scan, DNS resolution, and SYN stealth scan. Using `-vv` increases verbosity even further for additional detail.

---

## Banner Grabbing

Once the full scan completes, Nmap presents a table showing every open port alongside the service name and detected version.

**Command:**

```bash
sudo nmap 172.16.5.20 -p- -sV
```

**Output:**

```
Starting Nmap 7.80 ( https://nmap.org ) at 2020-06-15 20:00 CEST
Nmap scan report for 172.16.5.20
Host is up (0.013s latency).
Not shown: 65525 closed ports
PORT      STATE    SERVICE      VERSION
22/tcp    open     ssh          OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
25/tcp    open     smtp         Postfix smtpd
80/tcp    open     http         Apache httpd 2.4.29 ((Ubuntu))
110/tcp   open     pop3         Dovecot pop3d
139/tcp   filtered netbios-ssn
143/tcp   open     imap         Dovecot imapd (Ubuntu)
445/tcp   filtered microsoft-ds
993/tcp   open     ssl/imap     Dovecot imapd (Ubuntu)
995/tcp   open     ssl/pop3     Dovecot pop3d
MAC Address: DE:AD:00:00:BE:EF (Intel Corporate)
Service Info: Host:  inlane; OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 91.73 seconds
```

**Option Breakdown:**

| Option | Description |
|--------|-------------|
| 172.16.5.20 | The target being scanned |
| -p- | Scans all 65535 ports |
| -sV | Performs service and version detection |

The results immediately show useful details. SSH is running OpenSSH 7.6p1 on Ubuntu. The web server is Apache 2.4.29. Mail services are handled by Postfix for SMTP and Dovecot for POP3 and IMAP. The Service Info line at the bottom confirms the operating system is Linux.

Nmap collects most of this version information from service banners. When a connection is established to an open port, many services automatically send a greeting message that includes their name and version. Nmap reads that banner and records it. When a banner is not clear enough, Nmap falls back to a signature-based matching system, though this takes considerably longer. Some services are also configured to withhold or alter their banners, which means Nmap may not always capture the full picture.

---

## What Nmap May Miss

Even when Nmap performs version detection, there are situations where the raw service banner contains more information than Nmap displays in its final output. The following command uses packet tracing to expose exactly what data was exchanged during the service detection process.

**Command:**

```bash
sudo nmap 172.16.5.20 -p- -sV -Pn -n --disable-arp-ping --packet-trace
```

**Output:**

```
Starting Nmap 7.80 ( https://nmap.org ) at 2020-06-16 20:10 CEST
<SNIP>
NSOCK INFO [0.4200s] nsock_trace_handler_callback(): Callback: READ SUCCESS for EID 18 [172.16.5.20:25] (35 bytes): 220 inlane ESMTP Postfix (Ubuntu)..
Service scan match (Probe NULL matched with NULL line 3104): 172.16.5.20:25 is smtp.  Version: |Postfix smtpd|||
NSOCK INFO [0.4200s] nsock_iod_delete(): nsock_iod_delete (IOD #1)
Nmap scan report for 172.16.5.20
Host is up (0.076s latency).

PORT   STATE SERVICE VERSION
25/tcp open  smtp    Postfix smtpd
MAC Address: DE:AD:00:00:BE:EF (Intel Corporate)
Service Info: Host:  inlane

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 0.47 seconds
```

**Option Breakdown:**

| Option | Description |
|--------|-------------|
| 172.16.5.20 | The target being scanned |
| -p- | Scans all 65535 ports |
| -sV | Performs service and version detection |
| -Pn | Disables ICMP echo requests |
| -n | Disables DNS resolution |
| --disable-arp-ping | Disables ARP ping |
| --packet-trace | Shows all packets sent and received |

Looking at the packet trace line:

```
NSOCK INFO [0.4200s] nsock_trace_handler_callback(): Callback: READ SUCCESS for EID 18 [172.16.5.20:25] (35 bytes): 220 inlane ESMTP Postfix (Ubuntu)..
```

The raw banner the SMTP server sent was `220 inlane ESMTP Postfix (Ubuntu)`. This includes the Linux distribution name, Ubuntu. However, the final Nmap report only shows `Postfix smtpd` without mentioning Ubuntu. Nmap parsed the banner, matched it to a known signature, and recorded the service name but dropped part of the raw banner content in the process.

This is a known limitation of automated scanning. Nmap processes banners against its signature database and outputs what it can match, but it does not always preserve every detail the service sent. When full banner information is important, connecting directly to the service manually is the reliable way to get it.

---

## Manual Banner Grabbing with Netcat and Tcpdump

To capture the complete banner and observe the full network exchange, you can use netcat to connect directly to the service while tcpdump records everything happening on the wire.

**Start tcpdump first to capture traffic:**

```bash
sudo tcpdump -i eth0 host 192.168.1.10 and 172.16.5.20
```

**Output:**

```
tcpdump: verbose output suppressed, use -v or -vv for full protocol decode
listening on eth0, link-type EN10MB (Ethernet), capture size 262144 bytes
```

**Then connect to the SMTP port using netcat:**

```bash
nc -nv 172.16.5.20 25
```

**Output:**

```
Connection to 172.16.5.20 port 25 [tcp/*] succeeded!
220 inlane ESMTP Postfix (Ubuntu)
```

The full banner `220 inlane ESMTP Postfix (Ubuntu)` is immediately visible upon connection. The distribution name Ubuntu is present in the raw greeting that the server sends.

---

## Reading the Tcpdump Capture

While the netcat connection was active, tcpdump captured the complete exchange between the scanner (192.168.1.10) and the target (172.16.5.20).

**Captured Traffic:**

```
18:28:07.128564 IP 192.168.1.10.59618 > 172.16.5.20.smtp: Flags [S], seq 1798872233, win 65535, options [mss 1460,nop,wscale 6,nop,nop,TS val 331260178 ecr 0,sackOK,eol], length 0
18:28:07.255151 IP 172.16.5.20.smtp > 192.168.1.10.59618: Flags [S.], seq 1130574379, ack 1798872234, win 65160, options [mss 1460,sackOK,TS val 1800383922 ecr 331260178,nop,wscale 7], length 0
18:28:07.255281 IP 192.168.1.10.59618 > 172.16.5.20.smtp: Flags [.], ack 1, win 2058, options [nop,nop,TS val 331260304 ecr 1800383922], length 0
18:28:07.319306 IP 172.16.5.20.smtp > 192.168.1.10.59618: Flags [P.], seq 1:36, ack 1, win 510, options [nop,nop,TS val 1800383985 ecr 331260304], length 35: SMTP: 220 inlane ESMTP Postfix (Ubuntu)
18:28:07.319426 IP 192.168.1.10.59618 > 172.16.5.20.smtp: Flags [.], ack 36, win 2058, options [nop,nop,TS val 331260368 ecr 1800383985], length 0
```

The five lines in the capture represent a complete TCP connection sequence. Each line corresponds to a specific step.

**Step 1 - SYN:**

```
18:28:07.128564 IP 192.168.1.10.59618 > 172.16.5.20.smtp: Flags [S]
```

The scanner sends the first packet of the TCP three-way handshake. The `[S]` flag indicates this is a SYN packet, initiating the connection request from source port 59618 to the SMTP port on the target.

**Step 2 - SYN-ACK:**

```
18:28:07.255151 IP 172.16.5.20.smtp > 192.168.1.10.59618: Flags [S.]
```

The target responds with a SYN-ACK. The `[S.]` notation means both SYN and ACK flags are set. This tells the scanner that the target received the connection request and is willing to proceed.

**Step 3 - ACK:**

```
18:28:07.255281 IP 192.168.1.10.59618 > 172.16.5.20.smtp: Flags [.]
```

The scanner sends an ACK to complete the three-way handshake. The `[.]` flag represents ACK. At this point the TCP connection is fully established and data can flow in both directions.

**Step 4 - PSH-ACK (Banner Delivery):**

```
18:28:07.319306 IP 172.16.5.20.smtp > 192.168.1.10.59618: Flags [P.], length 35: SMTP: 220 inlane ESMTP Postfix (Ubuntu)
```

The target SMTP server sends its banner using a packet with PSH and ACK flags set. The PSH flag instructs the receiving system to pass the data immediately to the application rather than buffering it. The ACK flag confirms all previously received data. The 35 bytes of payload contain the full banner string including the operating system information that Nmap omitted in its summary output.

**Step 5 - ACK (Receipt Confirmation):**

```
18:28:07.319426 IP 192.168.1.10.59618 > 172.16.5.20.smtp: Flags [.]
```

The scanner sends a final ACK to confirm receipt of the banner data. This is standard TCP behavior acknowledging that the data was received successfully.

The tcpdump capture confirms that the target sent the Ubuntu detail as part of its banner at the network level. This information was present in the traffic but was not fully reflected in Nmap's final report, which is why manually connecting to services and capturing traffic is a reliable technique for extracting every detail a service exposes.
