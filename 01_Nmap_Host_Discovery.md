# Nmap Host Discovery

When conducting an internal penetration test across an entire company network, the first task is identifying which systems are online and reachable. Nmap provides a wide range of host discovery options for this purpose. The most reliable and commonly used method is sending ICMP echo requests, though other techniques exist depending on the environment.

Storing every scan result is considered best practice. Different tools can return different results, and having saved output allows for easy comparison, documentation, and reporting later on.

---

## Scanning a Network Range

The following command scans the entire subnet and returns only the IP addresses of live hosts by disabling port scanning and filtering the output.

**Command:**

```bash
sudo nmap 10.129.2.0/24 -sn -oA tnet | grep for | cut -d" " -f5
```

**Output:**

```
10.129.2.4
10.129.2.10
10.129.2.11
10.129.2.18
10.129.2.19
10.129.2.20
10.129.2.28
```

**Option Breakdown:**

| Option | Description |
|--------|-------------|
| 10.129.2.0/24 | The target network range to scan |
| -sn | Disables port scanning, only performs host discovery |
| -oA tnet | Saves results in all formats using the filename prefix "tnet" |

This scanning method works only if the firewalls of the hosts allow it. Otherwise, we can use other scanning techniques to find out if the hosts are active or not. We will take a closer look at these techniques in "Firewall and IDS Evasion".

---

## Scanning from an IP List

During internal engagements, a list of target IPs is often provided ahead of time. Nmap can read directly from such a list rather than requiring each IP to be typed manually.

A typical hosts file looks like this:

```bash
cat hosts.lst
```

```
10.129.2.4
10.129.2.10
10.129.2.11
10.129.2.18
10.129.2.19
10.129.2.20
10.129.2.28
```

**Command:**

```bash
sudo nmap -sn -oA tnet -iL hosts.lst | grep for | cut -d" " -f5
```

**Output:**

```
10.129.2.18
10.129.2.19
10.129.2.20
```

**Option Breakdown:**

| Option | Description |
|--------|-------------|
| -sn | Disables port scanning |
| -oA tnet | Saves results in all formats with the prefix "tnet" |
| -iL | Reads target IPs from the provided list file |

Only 3 of the 7 hosts responded. The remaining 4 did not reply, which likely means their firewalls are configured to drop ICMP echo requests. Nmap marks hosts without a reply as inactive by default.

---

## Scanning Multiple IPs

When only a few specific targets need to be scanned, individual IPs can be listed directly in the command rather than using a file.

**Command:**

```bash
sudo nmap -sn -oA tnet 10.129.2.18 10.129.2.19 10.129.2.20 | grep for | cut -d" " -f5
```

**Output:**

```
10.129.2.18
10.129.2.19
10.129.2.20
```

If the IPs fall within a continuous range, a shorthand notation using a hyphen can be used instead.

**Command:**

```bash
sudo nmap -sn -oA tnet 10.129.2.18-20 | grep for | cut -d" " -f5
```

**Output:**

```
10.129.2.18
10.129.2.19
10.129.2.20
```

Both commands produce identical results. The range notation is simply a faster way to write it when addresses are consecutive.

---

## Scanning a Single IP

Before attempting to enumerate ports or services on a host, it is important to verify the host is online. The same approach applies here, targeting one address at a time.

**Command:**

```bash
sudo nmap 10.129.2.18 -sn -oA host
```

**Output:**

```
Starting Nmap 7.80 ( https://nmap.org ) at 2020-06-14 23:59 CEST
Nmap scan report for 10.129.2.18
Host is up (0.087s latency).
MAC Address: DE:AD:00:00:BE:EF
Nmap done: 1 IP address (1 host up) scanned in 0.11 seconds
```

**Option Breakdown:**

| Option | Description |
|--------|-------------|
| 10.129.2.18 | The single target IP to scan |
| -sn | Disables port scanning |
| -oA host | Saves results in all formats with the prefix "host" |

When port scanning is disabled with `-sn`, Nmap defaults to using ICMP Echo Requests to determine whether a host is alive. However, on local networks, Nmap first sends an ARP ping before any ICMP traffic goes out. The ARP reply is enough to confirm the host is active. This behavior can be observed using packet tracing.

---

## Using Packet Trace to Observe ARP Behavior

To see exactly what packets Nmap is sending and receiving, the `--packet-trace` option can be added. The `-PE` flag explicitly requests ICMP Echo Requests, though on local networks ARP takes priority.

**Command:**

```bash
sudo nmap 10.129.2.18 -sn -oA host -PE --packet-trace
```

**Output:**

```
Starting Nmap 7.80 ( https://nmap.org ) at 2020-06-15 00:08 CEST
SENT (0.0074s) ARP who-has 10.129.2.18 tell 10.10.14.2
RCVD (0.0309s) ARP reply 10.129.2.18 is-at DE:AD:00:00:BE:EF
Nmap scan report for 10.129.2.18
Host is up (0.023s latency).
MAC Address: DE:AD:00:00:BE:EF
Nmap done: 1 IP address (1 host up) scanned in 0.05 seconds
```

**Option Breakdown:**

| Option | Description |
|--------|-------------|
| 10.129.2.18 | The target IP to scan |
| -sn | Disables port scanning |
| -oA host | Saves results with the prefix "host" |
| -PE | Instructs Nmap to use ICMP Echo Requests for the ping scan |
| --packet-trace | Displays every packet that is sent and received during the scan |

The output clearly shows that Nmap sent an ARP request rather than an ICMP packet, and the host replied with its MAC address. This is what confirmed the host as alive, not ICMP at all.

---

## Using the Reason Option

The `--reason` flag provides a plain-language explanation of why Nmap marked a host as alive or dead, without needing to read through raw packet data.

**Command:**

```bash
sudo nmap 10.129.2.18 -sn -oA host -PE --reason
```

**Output:**

```
Starting Nmap 7.80 ( https://nmap.org ) at 2020-06-15 00:10 CEST
SENT (0.0074s) ARP who-has 10.129.2.18 tell 10.10.14.2
RCVD (0.0309s) ARP reply 10.129.2.18 is-at DE:AD:00:00:BE:EF
Nmap scan report for 10.129.2.18
Host is up, received arp-response (0.028s latency).
MAC Address: DE:AD:00:00:BE:EF
Nmap done: 1 IP address (1 host up) scanned in 0.03 seconds
```

**Option Breakdown:**

| Option | Description |
|--------|-------------|
| 10.129.2.18 | The target IP to scan |
| -sn | Disables port scanning |
| -oA host | Saves results with the prefix "host" |
| -PE | Instructs Nmap to use ICMP Echo Requests |
| --reason | Displays the reason Nmap classified the host as up or down |

The line `Host is up, received arp-response` confirms that the detection was based on an ARP reply, not ICMP. This is the default behavior for hosts on the same local subnet.

---

## Forcing ICMP and Disabling ARP

To actually send ICMP Echo Requests and prevent ARP from taking over, ARP pings must be explicitly disabled using `--disable-arp-ping`. This is useful when testing how a host responds to ICMP specifically, or when working in environments where ARP may not be appropriate.

**Command:**

```bash
sudo nmap 10.129.2.18 -sn -oA host -PE --packet-trace --disable-arp-ping
```

**Output:**

```
Starting Nmap 7.80 ( https://nmap.org ) at 2020-06-15 00:12 CEST
SENT (0.0107s) ICMP [10.10.14.2 > 10.129.2.18 Echo request (type=8/code=0) id=13607 seq=0] IP [ttl=255 id=23541 iplen=28 ]
RCVD (0.0152s) ICMP [10.129.2.18 > 10.10.14.2 Echo reply (type=0/code=0) id=13607 seq=0] IP [ttl=128 id=40622 iplen=28 ]
Nmap scan report for 10.129.2.18
Host is up (0.086s latency).
MAC Address: DE:AD:00:00:BE:EF
Nmap done: 1 IP address (1 host up) scanned in 0.11 seconds
```

This time the output shows a genuine ICMP Echo Request being sent and an ICMP Echo Reply being received. The host confirmed its availability through ICMP rather than ARP. The TTL value in the reply (128) is also visible, which can be a useful indicator of the target operating system during later analysis.

