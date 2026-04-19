# Nmap Cheat Sheet

A practical reference for Nmap commands covering host discovery, scan types, service detection, scripting, output, performance, and evasion. Intended for use in lab environments, CTF challenges, and authorized penetration testing.

---

## Host Discovery

```bash
nmap -sn 192.168.56.0/24
```
Sends ping probes across the subnet to identify which hosts are online without scanning any ports.

```bash
nmap -Pn 192.168.56.101
```
Skips host discovery entirely and treats the target as online. Useful when ICMP is blocked by a firewall.

```bash
nmap -n 192.168.56.101
```
Disables DNS resolution. Nmap will not attempt to resolve hostnames, which speeds up the scan.

```bash
nmap -R 192.168.56.101
```
Forces reverse DNS resolution for all hosts, even those that appear offline.

---

## Basic Scanning

```bash
nmap 192.168.56.101
```
Runs a default scan against the top 1000 most common TCP ports.

```bash
nmap -F 192.168.56.101
```
Fast scan that checks only the top 100 ports instead of 1000.

```bash
nmap -p 80 192.168.56.101
```
Scans a single specific port.

```bash
nmap -p 1-1000 192.168.56.101
```
Scans a defined port range.

```bash
nmap -p- 192.168.56.101
```
Scans all 65535 TCP ports. Takes longer but ensures nothing is missed on non-standard ports.

```bash
nmap --top-ports 20 192.168.56.101
```
Scans only the 20 most commonly used ports.

---

## Scan Types

```bash
nmap -sS 192.168.56.101
```
SYN scan. Sends a SYN packet and waits for a response without completing the TCP handshake. Faster and less likely to be logged than a full connect scan. Requires root privileges.

```bash
nmap -sT 192.168.56.101
```
TCP connect scan. Completes the full three-way handshake. Used when raw socket access is unavailable.

```bash
nmap -sU 192.168.56.101
```
UDP scan. Slower than TCP scans because UDP does not respond to closed ports in a consistent way.

```bash
nmap -sA 192.168.56.101
```
ACK scan. Used to map firewall rules rather than discover open ports. Ports that respond are marked as unfiltered, meaning ACK packets are reaching the host.

```bash
nmap -sW 192.168.56.101
```
TCP window scan. Similar to the ACK scan but examines the TCP window field of the RST response to attempt to differentiate open from closed ports.

```bash
nmap -sM 192.168.56.101
```
Maimon scan. Sends FIN and ACK flags together. Works against certain older BSD-based systems.

---

## Service and OS Detection

```bash
nmap -sV 192.168.56.101
```
Probes open ports to identify the service name and version running on each one.

```bash
nmap -sV --version-intensity 5 192.168.56.101
```
Increases the intensity of version detection probes. Higher values are more thorough but slower and noisier.

```bash
nmap -O 192.168.56.101
```
Attempts to identify the operating system based on how the host responds to various TCP and IP probes.

```bash
nmap -A 192.168.56.101
```
Runs OS detection, version detection, default scripts, and traceroute together in a single command. Good for lab use when speed matters more than stealth.

---

## Nmap Scripting Engine

```bash
nmap -sC 192.168.56.101
```
Runs the default set of NSE scripts against the target. These scripts cover common enumeration and detection tasks.

```bash
nmap --script vuln 192.168.56.101
```
Runs scripts from the vulnerability category to check for known weaknesses in detected services.

```bash
nmap --script http-enum -p 80 192.168.56.101
```
Enumerates common web directories and files on a web server running on port 80.

```bash
nmap --script ftp-anon -p 21 192.168.56.101
```
Checks whether the FTP service allows anonymous login without credentials.

```bash
nmap --script smb-os-discovery -p 445 192.168.56.101
```
Queries the SMB service for operating system and hostname information.

---

## Input and Output

```bash
nmap -iL targets.txt
```
Reads scan targets from a text file instead of specifying them on the command line. One target per line.

```bash
nmap --exclude 192.168.56.1 192.168.56.0/24
```
Scans the entire subnet but skips the specified host.

```bash
nmap -oN scan.txt 192.168.56.101
```
Saves output to a file in normal readable format.

```bash
nmap -oX scan.xml 192.168.56.101
```
Saves output in XML format, which is useful for importing into other tools.

```bash
nmap -oA scan 192.168.56.101
```
Saves output simultaneously in normal, XML, and grepable formats. Recommended as a default habit during engagements.

---

## Performance and Timing

```bash
nmap -T4 192.168.56.101
```
Applies the aggressive timing template, which reduces timeouts and increases parallelism. Suitable for stable networks.

```bash
nmap --min-rate 1000 192.168.56.101
```
Sets a minimum packet sending rate of 1000 packets per second.

```bash
nmap --max-retries 2 192.168.56.101
```
Limits probe retries to 2 per port, reducing scan time at the cost of some reliability.

```bash
nmap --host-timeout 5m 192.168.56.101
```
Stops scanning a host if it has not completed within 5 minutes. Useful when scanning large ranges.

```bash
nmap --stats-every 10s 192.168.56.101
```
Prints scan progress to the terminal every 10 seconds.

Timing template reference:

| Template | Flag | Behavior |
|---|---|---|
| Paranoid | `-T0` | Extremely slow, minimal network noise |
| Sneaky | `-T1` | Slow, designed to avoid IDS detection |
| Polite | `-T2` | Reduced speed to avoid congestion |
| Normal | `-T3` | Default behavior |
| Aggressive | `-T4` | Faster, assumes a reliable network |
| Insane | `-T5` | Maximum speed, may miss results |

---

## Network Utilities

```bash
nmap --traceroute 192.168.56.101
```
Traces the network path to the target and displays each hop.

```bash
nmap --reason 192.168.56.101
```
Shows the reason Nmap assigned each port state, such as which flag was received in the response.

```bash
nmap -6 ::1
```
Enables IPv6 scanning mode.

---

## Evasion and Stealth

```bash
nmap -D RND:5 192.168.56.101
```
Decoy scan. Inserts five randomly generated IP addresses into packet headers alongside the real source IP, making it harder to identify the actual scanner. Decoy hosts should be alive for this to work correctly.

```bash
nmap -S 192.168.56.200 -e wlan0 192.168.56.101
```
Spoofs the source IP address. Useful for testing whether a firewall applies rules based on source subnet. Requires specifying the outgoing interface with `-e`.

```bash
nmap --source-port 53 192.168.56.101
```
Sends scan packets from source port 53. Some firewalls trust DNS traffic and will pass packets originating from this port even when other traffic is blocked.

---

## Recommended Workflow

This is a practical sequence for enumerating a target from scratch:

```bash
# Step 1: Quick scan to identify open ports and services
nmap -sV --open -oA initial_scan <target>

# Step 2: Full port scan to catch anything running on non-standard ports
nmap -p- --open -oA full_tcp_scan <target>

# Step 3: Run default scripts against confirmed open ports
nmap -sC -p <open_ports> -oA script_scan <target>

# Step 4: Web enumeration if port 80 or 443 is open
nmap -sV --script=http-enum -oA http_enum <target>
```

Run these in order. The initial scan gives you a quick picture of the target. The full port scan ensures nothing is hiding on an unusual port. The script scan digs into what you found. The web enumeration step is only relevant when a web server is present.

Always save your output. Output files are useful for review, reporting, and picking up where you left off.

---

## Usage Notice

Only scan systems you own or have explicit written permission to test. Unauthorized scanning is illegal in most jurisdictions and violates the terms of service of any network you do not control.
