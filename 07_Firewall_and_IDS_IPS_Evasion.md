# Firewall and IDS/IPS Evasion

Nmap provides several techniques to work around firewall rules and evade detection by IDS/IPS systems. These include packet fragmentation, decoy scanning, source port manipulation, and DNS proxying. Understanding how these defenses work is a prerequisite for using evasion techniques effectively.

---

## Firewalls

A firewall controls traffic between networks by enforcing a ruleset on every incoming and outgoing connection. Each packet is evaluated against these rules and is either allowed through, silently dropped, or explicitly rejected. Dropped packets receive no response, while rejected packets return an error to the sender. This distinction becomes important when interpreting Nmap scan results, particularly when ports appear as filtered.

## IDS/IPS

An intrusion detection system (IDS) passively monitors network traffic and alerts administrators when it identifies patterns that match known attack signatures. An intrusion prevention system (IPS) goes a step further by automatically taking action, such as blocking a connection, when a threat is detected. Both systems rely on signature and pattern matching, which means that certain Nmap scan types, particularly aggressive ones, can trigger them.

---

## Determining Firewall Rules

When a port appears as filtered during a scan, it means Nmap did not receive a usable response. This can happen because the firewall dropped the packet silently or because the host returned an ICMP error. Common ICMP error types in this context include:

- Net Unreachable
- Net Prohibited
- Host Unreachable
- Host Prohibited
- Port Unreachable
- Proto Unreachable

The ACK scan (`-sA`) is useful here because it behaves differently from a SYN or Connect scan. It sends a TCP packet with only the ACK flag set. Since ACK packets are associated with already-established connections, many firewalls pass them through without applying the same rules used against SYN packets. The host must respond with an RST flag regardless of whether the port is open or closed, which tells us whether the packet reached its destination.

### SYN Scan

```bash
sudo nmap 192.168.100.96 -p 21,22,25 -sS -Pn -n --disable-arp-ping --packet-trace
```

```
SENT (0.1290s) TCP 192.168.100.97:48396 > 192.168.100.96:25 S ttl=46 id=60265 iplen=44  seq=1653981580 win=1024 <mss 1460>
SENT (0.1291s) TCP 192.168.100.97:48396 > 192.168.100.96:21 S ttl=59 id=63062 iplen=44  seq=1653981580 win=1024 <mss 1460>
SENT (0.1291s) TCP 192.168.100.97:48396 > 192.168.100.96:22 S ttl=51 id=55703 iplen=44  seq=1653981580 win=1024 <mss 1460>
RCVD (0.1301s) TCP 192.168.100.96:25 > 192.168.100.97:48396 RA ttl=64 id=0 iplen=40  seq=0 win=0
RCVD (0.1301s) TCP 192.168.100.96:21 > 192.168.100.97:48396 RA ttl=64 id=0 iplen=40  seq=0 win=0
RCVD (0.1301s) TCP 192.168.100.96:22 > 192.168.100.97:48396 SA ttl=64 id=0 iplen=44  seq=1758886262 win=5840 <mss 1460>

PORT   STATE  SERVICE
21/tcp closed ftp
22/tcp open   ssh
25/tcp closed smtp
```

Port 22 returned a SYN-ACK (SA) response, confirming it is open and willing to complete the handshake. Ports 21 and 25 returned RST-ACK (RA), indicating they are closed. No packets were dropped here, so no firewall interference is visible on these ports.

### ACK Scan

```bash
sudo nmap 192.168.100.96 -p 21,22,25 -sA -Pn -n --disable-arp-ping --packet-trace
```

```
SENT (0.1072s) TCP 192.168.100.97:60548 > 192.168.100.96:21 A ttl=43 id=54081 iplen=40  seq=0 win=1024
SENT (0.1073s) TCP 192.168.100.97:60548 > 192.168.100.96:22 A ttl=57 id=47134 iplen=40  seq=0 win=1024
SENT (0.1073s) TCP 192.168.100.97:60548 > 192.168.100.96:25 A ttl=54 id=48246 iplen=40  seq=0 win=1024
RCVD (0.1078s) TCP 192.168.100.96:21 > 192.168.100.97:60548 R ttl=64 id=0 iplen=40  seq=2897143614 win=0
RCVD (0.1078s) TCP 192.168.100.96:22 > 192.168.100.97:60548 R ttl=64 id=0 iplen=40  seq=2897143614 win=0
RCVD (0.1078s) TCP 192.168.100.96:25 > 192.168.100.97:60548 R ttl=64 id=0 iplen=40  seq=2897143614 win=0

PORT   STATE      SERVICE
21/tcp unfiltered ftp
22/tcp unfiltered ssh
25/tcp unfiltered smtp
```

All three ports returned an RST, which means the ACK packets passed through the firewall and reached the host. The result of `unfiltered` does not tell us whether these ports are open or closed, but it confirms that no firewall is blocking ACK packets on this path. This is a useful baseline when mapping out what the firewall is actually enforcing.

| Option | Description |
|---|---|
| `192.168.100.96` | Target host |
| `-p 21,22,25` | Scan only these ports |
| `-sS` | SYN scan |
| `-sA` | ACK scan |
| `-Pn` | Skip host discovery |
| `-n` | Disable DNS resolution |
| `--disable-arp-ping` | Disable ARP ping |
| `--packet-trace` | Print all sent and received packets |

---

## Detecting IDS/IPS

IDS and IPS systems are harder to identify because they do not actively respond to packets. They sit passively on the network, inspecting traffic without directly interacting with the scanner. The most reliable way to detect their presence is by observing whether your access to the target is suddenly blocked after running certain scans.

The recommended approach during a penetration test is to scan from multiple VPS instances with different IP addresses. If one IP gets blocked mid-engagement, it is a strong indicator that an IPS is active and has flagged the traffic as hostile. At that point, switching to a different source IP and reducing scan aggressiveness is the appropriate response.

Certain behaviors, such as scanning a single port rapidly and repeatedly, are commonly used as test cases to trigger security responses. If the network reacts by cutting off access, the presence of automated protection can be confirmed.

---

## Decoys

In some networks, entire IP ranges are blocked at the perimeter, or an IPS is configured to block any host that sends too many connection attempts. The decoy method (`-D`) addresses this by injecting fake source IP addresses into the packet headers alongside the real one. From the target's perspective, connection attempts appear to be coming from multiple different hosts simultaneously, making it difficult to identify the actual scanner.

The `RND:5` argument tells Nmap to generate five random IP addresses. The real IP is inserted at a random position among them.

```bash
sudo nmap 192.168.100.96 -p 80 -sS -Pn -n --disable-arp-ping --packet-trace -D RND:5
```

```
SENT (0.0984s) TCP 153.238.79.200:56830 > 192.168.100.96:80 S ttl=55 id=45473 iplen=44  seq=454215460 win=1024 <mss 1460>
SENT (0.0985s) TCP 149.52.150.116:56830 > 192.168.100.96:80 S ttl=47 id=45473 iplen=44  seq=454215460 win=1024 <mss 1460>
SENT (0.0986s) TCP 192.168.100.97:56830 > 192.168.100.96:80 S ttl=42 id=45473 iplen=44  seq=454215460 win=1024 <mss 1460>
RCVD (0.1007s) TCP 192.168.100.96:80 > 192.168.100.97:56830 SA ttl=64 id=0 iplen=44  seq=2002722020 win=5840 <mss 1460>

PORT   STATE SERVICE
80/tcp open  http
```

The packet trace shows SYN packets being sent from several different IP addresses. Only the real source (`192.168.100.97`) receives the SYN-ACK back, since the other addresses are spoofed. Port 80 is confirmed open.

One important constraint: the decoy addresses need to be reachable hosts. If they are not, the target may become flooded with half-open connections from the spoofed IPs and fail to respond, which can cause the scan to produce inaccurate results.

| Option | Description |
|---|---|
| `192.168.100.96` | Target host |
| `-p 80` | Scan only port 80 |
| `-sS` | SYN scan |
| `-Pn` | Skip host discovery |
| `-n` | Disable DNS resolution |
| `--disable-arp-ping` | Disable ARP ping |
| `--packet-trace` | Print all sent and received packets |
| `-D RND:5` | Generate five random decoy IP addresses |

---

## Testing with a Different Source IP

Where firewall rules are based on source subnets rather than individual IPs, specifying a different source address with `-S` can help determine which subnets have access to which services. This is also useful for testing whether a specific IP range has been granted access to a service that is otherwise restricted.

**Default source scan:**

```bash
nmap 192.168.100.96 -n -Pn -p 445 -O
```

```
PORT    STATE SERVICE
445/tcp open  microsoft-ds

Running: Linux 2.6.X
OS details: Linux 2.6.17 - 2.6.36
```

**Scan using a spoofed source IP:**

```bash
sudo nmap 192.168.100.96 -n -Pn -p 445 -O -S 192.168.100.200 -e wlan0
```

```
PORT    STATE SERVICE
445/tcp open  microsoft-ds

Running: Linux 2.6.X
OS details: Linux 2.6.17 - 2.6.36
```

Both scans returned the same result, which tells us the firewall is not filtering based on source IP for this port. The OS fingerprint in both cases points to a Linux 2.6 kernel.

| Option | Description |
|---|---|
| `192.168.100.96` | Target host |
| `-n` | Disable DNS resolution |
| `-Pn` | Skip host discovery |
| `-p 445` | Scan only port 445 |
| `-O` | Enable OS detection |
| `-S 192.168.100.200` | Use a different source IP address |
| `-e wlan0` | Send packets through the specified interface |

---

## DNS Proxying

DNS queries are typically allowed through firewalls because blocking UDP port 53 would break normal name resolution for the network. TCP port 53, traditionally reserved for DNS zone transfers, is increasingly used for standard queries due to IPv6 and DNSSEC adoption. Many firewalls are configured to trust traffic on port 53 without close inspection.

Nmap allows you to set a custom source port with `--source-port`. If firewall rules trust TCP port 53, packets sent from that source port may pass through filters that would normally block them.

### SYN Scan on a Filtered Port

```bash
nmap 192.168.100.96 -p 50000 -sS -Pn -n --disable-arp-ping --packet-trace
```

```
SENT (0.0977s) TCP 192.168.100.97:37620 > 192.168.100.96:50000 S ttl=54 id=36331 iplen=44  seq=168281229 win=1024 <mss 1460>
RCVD (0.0982s) TCP 192.168.100.96:50000 > 192.168.100.97:37620 RA ttl=64 id=0 iplen=40  seq=0 win=0

PORT      STATE  SERVICE
50000/tcp closed ibm-db2
```

The port returned RST-ACK, which means it is reachable and closed. There is no firewall blocking traffic on this port from a standard source.

### SYN Scan Using Source Port 53

```bash
sudo nmap 192.168.100.96 -p 50000 -sS -Pn -n --disable-arp-ping --packet-trace --source-port 53
```

```
SENT (0.1000s) TCP 192.168.100.97:53 > 192.168.100.96:50000 S ttl=50 id=26038 iplen=44  seq=3126314855 win=1024 <mss 1460>
RCVD (0.1005s) TCP 192.168.100.96:50000 > 192.168.100.97:53 RA ttl=64 id=0 iplen=40  seq=0 win=0

PORT      STATE  SERVICE
50000/tcp closed ibm-db2
```

The result is the same here, but what matters is that the firewall allowed the packet through when it appeared to originate from port 53. In a scenario where the port was previously filtered, receiving any response after switching to source port 53 would confirm that the firewall is making trust decisions based on DNS port traffic.

| Option | Description |
|---|---|
| `192.168.100.96` | Target host |
| `-p 50000` | Scan only port 50000 |
| `-sS` | SYN scan |
| `-Pn` | Skip host discovery |
| `-n` | Disable DNS resolution |
| `--disable-arp-ping` | Disable ARP ping |
| `--packet-trace` | Print all sent and received packets |
| `--source-port 53` | Send packets from source port 53 |

Once it is confirmed that the firewall passes traffic from TCP port 53, the next step in a real engagement would be to connect directly using a tool like Netcat to verify whether the service behind that port is accessible through the same trust rule.
