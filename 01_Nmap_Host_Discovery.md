# Nmap Host Discovery

When conducting an internal penetration test across an entire company network, the first task is identifying which systems are online and reachable. Nmap provides a wide range of host discovery options for this purpose. The most reliable and commonly used method is sending ICMP echo requests, though other techniques exist depending on the environment.

Storing every scan result is considered best practice. Different tools can return different results, and having saved output allows for easy comparison, documentation, and reporting later on.

## Scanning a Network Range

The following command scans the entire subnet and returns only the IP addresses of live hosts by disabling port scanning and filtering the output.

**Command:**

```bash
sudo nmap 192.168.100.59/24 -sn -oA tnet | grep for | cut -d" " -f5
```

**Output:**

```
192.168.100.1
192.168.100.59
192.168.100.94
```

**Option Breakdown:**

| Option | Description |
|--------|-------------|
| 192.168.100.59/24 | The target network range to scan |
| -sn | Disables port scanning, only performs host discovery |
| -oA tnet | Saves results in all formats using the filename prefix "tnet" |

This scanning method works only if the firewalls of the hosts allow it. Otherwise, we can use other scanning techniques to find out if the hosts are active or not. We will take a closer look at these techniques in "Firewall and IDS Evasion".

## Scanning from an IP List

During internal engagements, a list of target IPs is often provided ahead of time. Nmap can read directly from such a list rather than requiring each IP to be typed manually.

A typical hosts file looks like this:

```bash
cat hosts.lst
```

```
192.168.100.1
192.168.100.59
192.168.100.94
```

**Command:**

```bash
sudo nmap -sn -oA tnet -iL hosts.lst | grep for | cut -d" " -f5
```

**Output:**

```
192.168.100.1
192.168.100.59
192.168.100.94
```

**Option Breakdown:**

| Option | Description |
|--------|-------------|
| -sn | Disables port scanning |
| -oA tnet | Saves results in all formats with the prefix "tnet" |
| -iL | Reads target IPs from the provided list file |

All 3 hosts in the list responded. If any of them had not replied, it would likely mean their firewall is configured to drop ICMP echo requests, and Nmap would mark those as inactive by default.

## Scanning Multiple IPs

When only a few specific targets need to be scanned, individual IPs can be listed directly in the command rather than using a file.

**Command:**

```bash
sudo nmap -sn -oA tnet 192.168.100.1 192.168.100.59 192.168.100.94 | grep for | cut -d" " -f5
```

**Output:**

```
192.168.100.1
192.168.100.59
192.168.100.94
```

If the IPs fall within a continuous range, a shorthand notation using a hyphen can be used instead.

**Command:**

```bash
sudo nmap -sn -oA tnet 192.168.100.0-200 | grep for | cut -d" " -f5
```

**Output:**

```
192.168.100.1
192.168.100.19
192.168.100.59
192.168.100.94
```

The range notation is simply a faster way to write it when addresses are consecutive. Note that scanning a wider range may return additional hosts that were not in the original list, as seen with 192.168.100.19 appearing here.

## Scanning a Single IP

Before attempting to enumerate ports or services on a host, it is important to verify the host is online. The same approach applies here, targeting one address at a time.

**Command:**

```bash
sudo nmap 192.168.100.59 -sn -oA host
```

**Output:**

```
Starting Nmap 7.98 ( https://nmap.org ) at 2026-04-02 23:31 -0400
Nmap scan report for 192.168.100.59
Host is up (0.00051s latency).
MAC Address: 08:00:27:00:14:D9 (Oracle VirtualBox virtual NIC)
Nmap done: 1 IP address (1 host up) scanned in 0.66 seconds
```

**Option Breakdown:**

| Option | Description |
|--------|-------------|
| 192.168.100.59 | The single target IP to scan |
| -sn | Disables port scanning |
| -oA host | Saves results in all formats with the prefix "host" |

When port scanning is disabled with `-sn`, Nmap defaults to using ICMP Echo Requests to determine whether a host is alive. However, on local networks, Nmap first sends an ARP ping before any ICMP traffic goes out. The ARP reply is enough to confirm the host is active. This behavior can be observed using packet tracing.

## Using Packet Trace to Observe ARP Behavior

To see exactly what packets Nmap is sending and receiving, the `--packet-trace` option can be added. The `-PE` flag explicitly requests ICMP Echo Requests, though on local networks ARP takes priority.

**Command:**

```bash
sudo nmap 192.168.100.59 -sn -oA host -PE --packet-trace
```

**Output:**

```
Starting Nmap 7.98 ( https://nmap.org ) at 2026-04-02 23:32 -0400
SENT (0.0359s) ARP who-has 192.168.100.59 tell 192.168.100.94
RCVD (0.0365s) ARP reply 192.168.100.59 is-at 08:00:27:00:14:D9
NSOCK INFO [0.0590s] nsock_iod_new2(): nsock_iod_new (IOD #1)
NSOCK INFO [0.0590s] nsock_connect_udp(): UDP connection requested to 192.168.100.1:53 (IOD #1) EID 8
NSOCK INFO [0.0590s] nsock_trace_handler_callback(): Callback: CONNECT SUCCESS for EID 8 [192.168.100.1:53]
NSOCK INFO [0.0590s] nsock_read(): Read request from IOD #1 [192.168.100.1:53] (timeout: -1ms) EID 18
NSOCK INFO [0.0590s] nsock_write(): Write request for 45 bytes to IOD #1 EID 27 [192.168.100.1:53]
NSOCK INFO [0.0590s] nsock_trace_handler_callback(): Callback: WRITE SUCCESS for EID 27 [192.168.100.1:53]
NSOCK INFO [0.0630s] nsock_trace_handler_callback(): Callback: READ SUCCESS for EID 18 [192.168.100.1:53] (45 bytes): N............59.100.168.192.in-addr.arpa.....
NSOCK INFO [0.0630s] nsock_read(): Read request from IOD #1 [192.168.100.1:53] (timeout: -1ms) EID 34
NSOCK INFO [0.5610s] nsock_iod_delete(): nsock_iod_delete (IOD #1)
NSOCK INFO [0.5610s] nevent_delete(): nevent_delete on event #34 (type READ)
Nmap scan report for 192.168.100.59
Host is up (0.00062s latency).
MAC Address: 08:00:27:00:14:D9 (Oracle VirtualBox virtual NIC)
Nmap done: 1 IP address (1 host up) scanned in 0.66 seconds
```

**Option Breakdown:**

| Option | Description |
|--------|-------------|
| 192.168.100.59 | The target IP to scan |
| -sn | Disables port scanning |
| -oA host | Saves results with the prefix "host" |
| -PE | Instructs Nmap to use ICMP Echo Requests for the ping scan |
| --packet-trace | Displays every packet that is sent and received during the scan |

The output clearly shows that Nmap sent an ARP request rather than an ICMP packet, and the host replied with its MAC address. This is what confirmed the host as alive, not ICMP at all.

## Using the Reason Option

The `--reason` flag explains why Nmap considers a host to be up or down without requiring analysis of raw packet data.

Command:

```bash
sudo nmap 10.129.2.18 -sn -oA host -PE --reason
```

Output:

```
Starting Nmap 7.98 ( https://nmap.org ) at 2026-04-02 23:33 -0400
SENT (0.0074s) ARP who-has 10.129.2.18 tell 10.10.14.2
RCVD (0.0309s) ARP reply 10.129.2.18 is-at 08:00:27:00:14:D9
Nmap scan report for 10.129.2.18
Host is up, received arp-response (0.028s latency).
MAC Address: 08:00:27:00:14:D9 (Oracle VirtualBox virtual NIC)
Nmap done: 1 IP address (1 host up) scanned in 0.03 seconds
```

Option Breakdown:

| Option | Description |
|--------|-------------|
| 10.129.2.18 | Target IP address |
| -sn | Disables port scanning (host discovery only) |
| -oA host | Saves output in all formats with the prefix "host" |
| -PE | Requests ICMP Echo probes |
| --reason | Displays why Nmap marked the host as up or down |

The line `Host is up, received arp-response` shows that the host was discovered using ARP instead of ICMP.

This happens because the target is on the same local network. In such cases, Nmap automatically prefers ARP requests since they are faster and more reliable than ICMP.

Note: On local networks, Nmap prioritizes ARP for host discovery. For external targets, ARP is not used and Nmap relies on ICMP or other probe techniques.

## Forcing ICMP and Disabling ARP

To force Nmap to use ICMP instead of ARP, ARP-based discovery must be disabled:

```bash
sudo nmap 10.129.2.18 -sn -PE --disable-arp-ping
```

This is useful when you want to test ICMP behavior or simulate how a host responds from an external network perspective.

## Forcing ICMP with Packet Trace

Command:

```bash
sudo nmap 10.129.2.18 -sn -oA host -PE --packet-trace --disable-arp-ping
```

Output:

```
Starting Nmap 7.98 ( https://nmap.org ) at 2026-04-02 23:34 -0400
SENT (0.0107s) ICMP [10.10.14.2 > 10.129.2.18 Echo request (type=8/code=0) id=13607 seq=0] IP [ttl=255 id=23541 iplen=28 ]
RCVD (0.0152s) ICMP [10.129.2.18 > 10.10.14.2 Echo reply (type=0/code=0) id=13607 seq=0] IP [ttl=128 id=40622 iplen=28 ]
Nmap scan report for 10.129.2.18
Host is up (0.086s latency).
MAC Address: 08:00:27:00:14:D9 (Oracle VirtualBox virtual NIC)
Nmap done: 1 IP address (1 host up) scanned in 0.11 seconds
```

Option Breakdown:

| Option | Description |
|--------|-------------|
| 10.129.2.18 | Target IP address |
| -sn | Disables port scanning |
| -oA host | Saves results with the prefix "host" |
| -PE | Sends ICMP Echo Requests |
| --packet-trace | Displays all packets sent and received |
| --disable-arp-ping | Disables ARP to force ICMP usage |

In this case, Nmap sends an ICMP Echo Request and receives an Echo Reply from the target.

Since ARP is disabled, Nmap cannot use its default local network behavior and instead uses ICMP for host discovery.

The TTL value in the reply (128) can provide a rough indication of the operating system. For example, many Windows systems use a default TTL close to 128, although this is not definitive.
