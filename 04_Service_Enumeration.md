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
Starting Nmap 7.80 ( https://nmap.org ) at 2026-04-01 19:44 CEST
Stats: 0:00:03 elapsed; 0 hosts completed (1 up), 1 undergoing SYN Stealth Scan
SYN Stealth Scan Timing: About 3.64% done; ETC: 19:45 (0:00:53 remaining)
```

---

### Automatic Progress Updates

**Command:**

```bash
sudo nmap 172.16.5.20 -p- -sV --stats-every=5s
```

**Output:**

```
Starting Nmap 7.80 ( https://nmap.org ) at 2026-04-01 19:46 CEST
Stats: 0:00:05 elapsed; 0 hosts completed (1 up), 1 undergoing SYN Stealth Scan
SYN Stealth Scan Timing: About 13.91% done; ETC: 19:49 (0:00:31 remaining)
Stats: 0:00:10 elapsed; 0 hosts completed (1 up), 1 undergoing SYN Stealth Scan
SYN Stealth Scan Timing: About 39.57% done; ETC: 19:48 (0:00:15 remaining)
```

---

### Verbose Mode for Live Port Discovery

**Command:**

```bash
sudo nmap 172.16.5.20 -p- -sV -v
```

**Output:**

```
Starting Nmap 7.80 ( https://nmap.org ) at 2026-04-01 20:03 CEST
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
