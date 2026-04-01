# Host and Port Scanning

Understanding how Nmap works internally is just as important as knowing which commands to run. The results only make sense when you understand what they mean and how they were obtained. Once a target has been confirmed as alive, the next step is building a more complete picture of the system. At this stage, the information we are looking for includes open ports and their associated services, service versions, any information the services expose, and the operating system running on the target.

---

## Port States

Nmap classifies every scanned port into one of six possible states. Knowing what each state means helps you make informed decisions about what to investigate further.

| State | Description |
|-------|-------------|
| open | A connection to the port was successfully established. This applies to TCP connections, UDP datagrams, and SCTP associations. |
| closed | The target responded with a TCP packet containing the RST flag, meaning the port is reachable but nothing is listening on it. This can also be used to confirm a host is alive. |
| filtered | Nmap could not determine whether the port is open or closed. Either no response came back, or an error code was returned. A firewall is usually responsible for this. |
| unfiltered | Only seen during a TCP-ACK scan. The port is accessible but it cannot be determined if it is open or closed. |
| open\|filtered | No response was received for the port. This typically means a firewall or packet filter is protecting it. |
| closed\|filtered | Only occurs during IP ID idle scans. It was impossible to tell if the port is closed or filtered by a firewall. |

---

## Discovering Open TCP Ports

By default, Nmap scans the top 1000 TCP ports using a SYN scan (-sS). This SYN scan only runs as default when Nmap is executed as root, because creating raw TCP packets requires elevated socket permissions. Without root, Nmap falls back to a full TCP connect scan (-sT) automatically.

Ports can be specified in several ways depending on what you need. You can list them individually with `-p 22,25,80`, define a range with `-p 22-445`, use the most frequently seen ports from the Nmap database with `--top-ports=10`, scan all 65535 ports with `-p-`, or run a fast scan of the top 100 ports using `-F`.

---

### Scanning Top 10 TCP Ports

**Command:**

```bash
sudo nmap 192.168.100.59 --top-ports=10
```

**Output:**

```
Starting Nmap 7.98 ( https://nmap.org ) at 2026-04-01 13:27 -0400
Nmap scan report for 192.168.100.59
Host is up (0.00080s latency).

PORT     STATE  SERVICE
21/tcp   closed ftp
22/tcp   open   ssh
23/tcp   closed telnet
25/tcp   closed smtp
80/tcp   open   http
110/tcp  closed pop3
139/tcp  open   netbios-ssn
443/tcp  open   https
445/tcp  open   microsoft-ds
3389/tcp closed ms-wbt-server
MAC Address: 08:00:27:00:14:D9 (Oracle VirtualBox virtual NIC)

Nmap done: 1 IP address (1 host up) scanned in 0.80 seconds
```

**Option Breakdown:**

| Option | Description |
|--------|-------------|
| 192.168.100.59 | The target being scanned |
| --top-ports=10 | Scans the 10 most commonly used TCP ports as defined in the Nmap database |

The output shows three different states across the ten ports. Ports 22, 25, 80, and 110 are open and have active services listening. Ports 21, 23, 443, and 3389 are closed, meaning the host responded with RST to confirm the port is not in use. Ports 139 and 445 are filtered, meaning no response came back, which typically points to a firewall dropping those packets.

---

### Tracing Packets on a Closed Port

To understand what actually happens when a port is closed, we can trace the packets Nmap sends and receives. The following command scans port 21 and disables ICMP echo requests, DNS resolution, and ARP ping so the SYN scan behavior is clear and unobstructed.

**Command:**

```bash
sudo nmap 192.168.100.59 -p 21 --packet-trace -Pn -n --disable-arp-ping
```

**Output:**

```
Starting Nmap 7.98 ( https://nmap.org ) at 2026-04-01 13:28 -0400
SENT (0.0980s) TCP 192.168.100.94:44093 > 192.168.100.59:21 S ttl=39 id=22759 iplen=44  seq=520204497 win=1024 <mss 1460>
RCVD (0.0991s) TCP 192.168.100.59:21 > 192.168.100.94:44093 RA ttl=64 id=0 iplen=40  seq=0 win=0 
Nmap scan report for 192.168.100.59
Host is up (0.0012s latency).

PORT   STATE  SERVICE
21/tcp closed ftp
MAC Address: 40:00:40:06:F0:E5 (Unknown)

Nmap done: 1 IP address (1 host up) scanned in 0.24 seconds

```

**Option Breakdown:**

| Option | Description |
|--------|-------------|
| 192.168.100.59 | The target being scanned |
| -p 21 | Scans only port 21 |
| --packet-trace | Shows all packets sent and received |
| -n | Disables DNS resolution |
| --disable-arp-ping | Disables ARP ping |

**Reading the Packet Trace:**

The SENT line shows that Nmap sent a TCP packet from its own IP (192.168.1.10) using source port 63090, directed at port 21 on the target. The `S` in the packet details indicates the SYN flag was set. This is the opening move of a TCP handshake.

The RCVD line shows the target replied with a packet containing the `RA` flags, which stands for RST and ACK. RST signals that no service is listening on that port and the connection attempt is being rejected. ACK acknowledges the receipt of the original SYN. Together they tell Nmap the port is definitively closed.

**Request breakdown:**

| Field | Meaning |
|-------|---------|
| SENT (0.0.0980s) | Nmap sent this packet at 0.0429 seconds into the scan |
| TCP | The protocol being used |
| 192.168.1.10:44093 > | Source IP and port (the scanner) |
| 192.168.100.59:21 | Destination IP and port (the target) |
| S | SYN flag set in the TCP header |
| ttl=39 id=22759 iplen=44  seq=520204497 win=1024 <mss 1460> | Additional TCP and IP header values |

**Response breakdown:**

| Field | Meaning |
|-------|---------|
| RCVD (0.0573s) | This packet was received at 0.0573 seconds |
| TCP | The protocol used in the response |
| 192.168.100.59:21 > | Source IP and port (the target responding) |
| 192.168.1.10:63090 | Destination IP and port (back to the scanner) |
| RA | RST and ACK flags set, indicating the port is closed |
| ttl=64 id=0 iplen=40  seq=0 win=0 | TCP and IP header values in the reply |

---

## Connect Scan

The TCP Connect Scan (-sT) performs a full three-way handshake to determine port state. It sends a SYN packet and waits for a response. If the target replies with SYN-ACK, the port is open. If it replies with RST, the port is closed. Nmap then sends a proper FIN to close the connection cleanly.

This scan is highly accurate because it goes through the complete handshake. The tradeoff is stealth. A full connection is established, which means most operating systems and applications will log the event. Modern IDS and IPS solutions detect this scan easily. However, it is still useful in environments where accuracy matters more than staying undetected, and it is considered a cleaner method because it behaves exactly like a legitimate client connection, causing minimal disruption to services.

The SYN scan (-sS) is considered more stealthy by comparison because it never completes the handshake. After receiving the SYN-ACK from an open port, it sends a RST instead of completing the connection. This leaves the connection half-open and reduces the likelihood of triggering logs, though advanced security systems have adapted to detect this behavior too.

**Command:**

```bash
sudo nmap 192.168.100.59 -p 443 --packet-trace --disable-arp-ping -Pn -n --reason -sT
```

**Output:**

```
Starting Nmap 7.98 ( https://nmap.org ) at 2026-04-01 13:34 -0400
CONN (0.0867s) TCP localhost > 192.168.100.59:443 => Operation now in progress
CONN (0.0873s) TCP localhost > 192.168.100.59:443 => Connected
Nmap scan report for 192.168.100.59
Host is up, received user-set (0.00054s latency).

PORT    STATE SERVICE REASON
443/tcp open  https   syn-ack

Nmap done: 1 IP address (1 host up) scanned in 0.09 seconds

```

The CONN lines in the packet trace show the full connection being established. First the connection is initiated (Operation now in progress), and then successfully completed (Connected). The reason field shows `syn-ack`, confirming the target responded with SYN-ACK to indicate port 443 is open.

---

## Filtered Ports

A filtered port means Nmap could not definitively determine whether it is open or closed. This usually happens when a firewall is involved. Firewalls can handle packets in two different ways: they can drop them silently, or they can actively reject them.

**When packets are dropped:**

When a firewall drops packets, Nmap receives no response at all. By default, Nmap retries up to 10 times (controlled by `--max-retries`) before giving up and marking the port as filtered. This is why scans involving filtered ports take much longer than scans where the target responds.

**Command:**

```bash
sudo nmap 192.168.100.59 -p 139 --packet-trace -n --disable-arp-ping -Pn
```

**Output:**

```
Starting Nmap 7.98 ( https://nmap.org ) at 2026-04-01 13:36 -0400
SENT (0.0950s) TCP 192.168.100.94:33892 > 192.168.100.59:139 S ttl=41 id=49086 iplen=44  seq=1868536983 win=1024 <mss 1460>
RCVD (0.0955s) TCP 192.168.100.59:139 > 192.168.100.94:33892 SA ttl=64 id=0 iplen=44  seq=3783376353 win=5840 <mss 1460>
Stats: 0:00:00 elapsed; 0 hosts completed (1 up), 1 undergoing SYN Stealth Scan
SYN Stealth Scan Timing: About 100.00% done; ETC: 13:36 (0:00:00 remaining)
Nmap scan report for 192.168.100.59
Host is up (0.00052s latency).

PORT    STATE SERVICE
139/tcp open  netbios-ssn
MAC Address: 40:00:40:06:F0:E1 (Unknown)

Nmap done: 1 IP address (1 host up) scanned in 0.24 seconds

```

**Option Breakdown:**

| Option | Description |
|--------|-------------|
| 192.168.100.59 | The target being scanned |
| -p 139 | Scans only port 139 |
| --packet-trace | Shows all packets sent and received |
| -n | Disables DNS resolution |
| --disable-arp-ping | Disables ARP ping |
| -Pn | Disables ICMP echo requests |

Two SYN packets were sent (at 0.0381s and 1.0411s) but no response was received for either. The total scan time of 2.06 seconds is noticeably longer compared to the closed port scan which completed in about 0.05 seconds. That extra time is Nmap waiting out the timeout period after each unanswered packet before trying again.

**When packets are rejected:**

When a firewall actively rejects a packet, it sends back an ICMP error message instead of silently dropping it. This still results in the port being marked filtered, but the scan completes much faster because Nmap receives an explicit response.

**Command:**

```bash
sudo nmap 192.168.100.59 -p 445 --packet-trace -n --disable-arp-ping -Pn
```

**Output:**

```
Starting Nmap 7.98 ( https://nmap.org ) at 2026-04-01 13:37 -0400
SENT (0.0975s) TCP 192.168.100.94:57559 > 192.168.100.59:445 S ttl=54 id=46801 iplen=44  seq=1207954545 win=1024 <mss 1460>
RCVD (0.0984s) TCP 192.168.100.59:445 > 192.168.100.94:57559 SA ttl=64 id=0 iplen=44  seq=632388572 win=5840 <mss 1460>
Nmap scan report for 192.168.100.59
Host is up (0.00089s latency).

PORT    STATE SERVICE
445/tcp open  microsoft-ds
MAC Address: 40:00:40:06:F0:E1 (Unknown)

Nmap done: 1 IP address (1 host up) scanned in 0.22 seconds

```

**Option Breakdown:**

| Option | Description |
|--------|-------------|
| 192.168.100.59 | The target being scanned |
| -p 445 | Scans only port 445 |
| --packet-trace | Shows all packets sent and received |
| -n | Disables DNS resolution |
| --disable-arp-ping | Disables ARP ping |
| -Pn | Disables ICMP echo requests |

This time a response came back, but it was an ICMP message with type 3 and code 3, which means "Port Unreachable". This is an active rejection by the firewall rather than a silent drop. The port is still marked as filtered, but the scan finished in 0.05 seconds because Nmap received an answer immediately. Knowing the host is alive and that a specific port is actively rejected confirms that a firewall rule is in place for that port, which is worth investigating further.

---

## Discovering Open UDP Ports

UDP scanning is a different challenge compared to TCP. UDP is a stateless protocol, meaning there is no handshake and no guaranteed acknowledgment. When Nmap sends a UDP packet to a port, it may or may not receive anything back. This makes it much harder to determine whether a port is actually open or simply not responding.

Because of this, UDP scans (-sU) have a much longer timeout period per port, and the overall scan takes significantly more time than a TCP scan. Scanning the top 100 UDP ports with `-F` can still take well over a minute.

**Command:**

```bash
sudo nmap 192.168.100.59 -F -sU
```

**Output:**

```
Starting Nmap 7.98 ( https://nmap.org ) at 2026-04-01 13:38 -0400
Nmap scan report for 192.168.100.59
Host is up (0.00057s latency).
Not shown: 58 closed udp ports (port-unreach), 41 open|filtered udp ports (no-response)
PORT    STATE SERVICE
137/udp open  netbios-ns
MAC Address: 08:00:27:00:14:D9 (Oracle VirtualBox virtual NIC)

Nmap done: 1 IP address (1 host up) scanned in 59.33 seconds
```

**Option Breakdown:**

| Option | Description |
|--------|-------------|
| 192.168.100.59 | The target being scanned |
| -F | Scans the top 100 ports |
| -sU | Performs a UDP scan |

The scan took 98 seconds for just 100 ports. Port 137 and 5353 returned definitive open results because the running services sent back a UDP response. Ports 68, 138, and 631 are marked `open|filtered` because no response came back, and Nmap cannot tell whether that is because they are open and not configured to reply, or because something is blocking the packet.

---

### UDP Port Open (with Response)

**Command:**

```bash
sudo nmap 192.168.100.59 -sU -Pn -n --disable-arp-ping --packet-trace -p 137 --reason
```

**Output:**

```
Starting Nmap 7.98 ( https://nmap.org ) at 2026-04-01 13:39 -0400
SENT (0.4090s) UDP 192.168.100.94:59643 > 192.168.100.59:137 ttl=53 id=63034 iplen=78 
SENT (0.4091s) UDP 192.168.100.94:59643 > 192.168.100.59:137 ttl=55 id=63034 iplen=78 
SENT (0.4091s) UDP 192.168.100.94:59643 > 192.168.100.59:137 ttl=41 id=63034 iplen=78 
RCVD (0.4101s) UDP 192.168.100.59:137 > 192.168.100.94:59643 ttl=64 id=0 iplen=365 
Nmap scan report for 192.168.100.59
Host is up, received user-set (0.0012s latency).

PORT    STATE SERVICE    REASON
137/udp open  netbios-ns udp-response ttl 64

Nmap done: 1 IP address (1 host up) scanned in 0.44 seconds

```

**Option Breakdown:**

| Option | Description |
|--------|-------------|
| 192.168.100.59 | The target being scanned |
| -sU | Performs a UDP scan |
| -Pn | Disables ICMP echo requests |
| -n | Disables DNS resolution |
| --disable-arp-ping | Disables ARP ping |
| --packet-trace | Shows all packets sent and received |
| -p 137 | Scans only port 137 |
| --reason | Displays the reason for the port state |

A UDP packet was sent and a UDP reply came back from the target. The reason shown is `udp-response`, which is the clearest possible confirmation that a service is actively listening and responding on that port. Port 137 is marked as definitively open.

---

### UDP Port Closed (ICMP Unreachable)

**Command:**

```bash
sudo nmap 192.168.100.59 -sU -Pn -n --disable-arp-ping --packet-trace -p 100 --reason
```

**Output:**

```
Starting Nmap 7.98 ( https://nmap.org ) at 2026-04-01 13:40 -0400
SENT (0.4095s) UDP 192.168.100.94:60808 > 192.168.100.59:100 ttl=37 id=1171 iplen=28 
RCVD (0.4099s) ICMP [192.168.100.59 > 192.168.100.94 Port unreachable (type=3/code=3) ] IP [ttl=64 id=1349 iplen=56 ]
Nmap scan report for 192.168.100.59
Host is up, received user-set (0.00043s latency).

PORT    STATE  SERVICE REASON
100/udp closed unknown port-unreach ttl 64

Nmap done: 1 IP address (1 host up) scanned in 0.45 seconds

```

A UDP packet was sent to port 100, and the target responded with an ICMP type 3, code 3 message meaning "Port Unreachable". This is how a system signals that nothing is listening on a UDP port. The reason shown is `port-unreach`, and the port is marked as closed.

---

### UDP Port Open|Filtered (No Response)

**Command:**

```bash
sudo nmap 192.168.100.59 -sU -Pn -n --disable-arp-ping --packet-trace -p 138 --reason
```

**Output:**

```
Starting Nmap 7.98 ( https://nmap.org ) at 2026-04-01 13:40 -0400
SENT (0.4140s) UDP 192.168.100.94:48404 > 192.168.100.59:138 ttl=38 id=40645 iplen=28 
SENT (1.4144s) UDP 192.168.100.94:48406 > 192.168.100.59:138 ttl=52 id=39660 iplen=28 
Nmap scan report for 192.168.100.59
Host is up, received user-set.

PORT    STATE         SERVICE     REASON
138/udp open|filtered netbios-dgm no-response

Nmap done: 1 IP address (1 host up) scanned in 2.45 seconds

```

**Option Breakdown:**

| Option | Description |
|--------|-------------|
| 192.168.100.59 | The target being scanned |
| -sU | Performs a UDP scan |
| -Pn | Disables ICMP echo requests |
| -n | Disables DNS resolution |
| --disable-arp-ping | Disables ARP ping |
| --packet-trace | Shows all packets sent and received |
| -p 138 | Scans only port 138 |
| --reason | Displays the reason for the port state |

Two packets were sent and nothing came back. The reason shown is `no-response`. Because UDP gives no inherent acknowledgment, silence could mean the port is open but the service does not reply to empty probes, or it could mean a firewall is dropping the packets. Nmap cannot tell the difference, so it marks the port `open|filtered`. Any ICMP response other than type 3 code 3 would also produce this same state.

---

## Version Scanning

Once open ports have been identified, the next step is finding out exactly what software is running on them. The `-sV` option sends additional probes after establishing a connection to pull version information, service names, and other details from open ports.

**Command:**

```bash
sudo nmap 192.168.100.59 -Pn -n --disable-arp-ping --packet-trace -p 445 --reason -sV
```

**Output:**

```
Starting Nmap 7.98 ( https://nmap.org ) at 2026-04-01 13:41 -0400
SENT (0.4494s) TCP 192.168.100.94:47699 > 192.168.100.59:445 S ttl=41 id=42238 iplen=44  seq=2765008816 win=1024 <mss 1460>
RCVD (0.4499s) TCP 192.168.100.59:445 > 192.168.100.94:47699 SA ttl=64 id=0 iplen=44  seq=3881286099 win=5840 <mss 1460>
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

**Option Breakdown:**

| Option | Description |
|--------|-------------|
| 192.168.100.59 | The target being scanned |
| -Pn | Disables ICMP echo requests |
| -n | Disables DNS resolution |
| --disable-arp-ping | Disables ARP ping |
| --packet-trace | Shows all packets sent and received |
| -p 445 | Scans only port 445 |
| --reason | Displays the reason for the port state |
| -sV | Performs service and version detection |

**What the packet trace reveals:**

The scan starts with a standard SYN packet. The target replies with SA (SYN-ACK), confirming the port is open. Nmap then establishes a full TCP connection and sends a NULL probe first, waiting up to 6 seconds for a response. When nothing comes back (READ TIMEOUT), it moves on to a more specific probe called SMBProgNeg, which is a known SMB protocol negotiation packet. The target replies with 135 bytes of data, which Nmap matches against its service signature database.

The final result identifies the service as Samba smbd running version 3.X to 4.X in a workgroup called WORKGROUP. The host is also identified as running Ubuntu. This level of detail is gathered without running any separate OS detection scan, simply by observing how the service responds to crafted probes.

Version scanning takes longer than a basic port scan because of the additional probe-and-response cycles, but the information it returns is far more actionable and directly useful for identifying potential vulnerabilities on the target.
