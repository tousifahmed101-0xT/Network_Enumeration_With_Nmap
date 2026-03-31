# Saving the Results

Every scan you run during a penetration test should be saved. Going back to earlier results lets you compare different scanning methods, track changes over time, and build accurate documentation for your final report. Nmap supports three output formats, each suited to a different purpose. You can save to any one of them individually or write all three at the same time.

The three formats are:

Normal output (-oN) saved with a .nmap file extension, grepable output (-oG) saved with a .gnmap file extension, and XML output (-oX) saved with a .xml file extension.

The option (-oA) writes all three formats simultaneously using a single filename prefix, which is the most practical choice during a real engagement.

---

## Saving All Formats at Once

**Command:**

```bash
sudo nmap 192.168.2.15 -p- -oA target
```

**Output:**

```
Starting Nmap 7.80 ( https://nmap.org ) at 2020-06-16 12:14 CEST
Nmap scan report for 192.168.2.15
Host is up (0.0091s latency).
Not shown: 65525 closed ports
PORT      STATE SERVICE
22/tcp    open  ssh
25/tcp    open  smtp
80/tcp    open  http
MAC Address: DE:AD:00:00:BE:EF (Intel Corporate)

Nmap done: 1 IP address (1 host up) scanned in 10.22 seconds
```

**Option Breakdown:**

| Option | Description |
|--------|-------------|
| 192.168.2.15 | The target being scanned |
| -p- | Scans all 65535 ports |
| -oA target | Saves results in all three formats, naming each file with the prefix "target" |

When no full path is provided, Nmap saves the files in whatever directory you are currently working in. After the scan completes, three files will be present.

**Command:**

```bash
ls
```

**Output:**

```
target.gnmap target.xml  target.nmap
```

---

## Normal Output

The normal output file is the most readable format. It mirrors exactly what Nmap prints to the terminal during a scan, making it the easiest format to review quickly or share in plain text form.

**Command:**

```bash
cat target.nmap
```

**Output:**

```
# Nmap 7.80 scan initiated Tue Jun 16 12:14:53 2020 as: nmap -p- -oA target 192.168.2.15
Nmap scan report for 192.168.2.15
Host is up (0.053s latency).
Not shown: 4 closed ports
PORT   STATE SERVICE
22/tcp open  ssh
25/tcp open  smtp
80/tcp open  http
MAC Address: DE:AD:00:00:BE:EF (Intel Corporate)

# Nmap done at Tue Jun 16 12:15:03 2020 -- 1 IP address (1 host up) scanned in 10.22 seconds
```

The file includes a comment at the top showing the exact command that was used and the timestamp when the scan started. At the bottom there is a closing comment with the completion time and a summary. This makes it easy to reconstruct the context of any scan just by reading the file, which is valuable when reviewing notes days or weeks later.

---

## Grepable Output

The grepable format compresses all information about each host onto a single line. This makes it straightforward to extract specific data using standard command-line tools like grep, awk, and cut without needing to parse multi-line blocks.

**Command:**

```bash
cat target.gnmap
```

**Output:**

```
# Nmap 7.80 scan initiated Tue Jun 16 12:14:53 2020 as: nmap -p- -oA target 192.168.2.15
Host: 192.168.2.15 ()    Status: Up
Host: 192.168.2.15 ()    Ports: 22/open/tcp//ssh///, 25/open/tcp//smtp///, 80/open/tcp//http///  Ignored State: closed (4)
# Nmap done at Tue Jun 16 12:14:53 2020 -- 1 IP address (1 host up) scanned in 10.22 seconds
```

Each open port is recorded in a consistent format: port number, state, protocol, service name, separated by forward slashes. This structure is what makes it easy to filter. For example, if you wanted to pull every host with port 80 open from a large scan result, a single grep command against the .gnmap file would give you that list instantly. This format is especially useful when dealing with large networks where you need to sort and filter hundreds of results quickly.

---

## XML Output

The XML format is the most detailed and structured of the three. It stores every piece of information Nmap collected, including timing data, detection reasons, TTL values, MAC addresses, and service confidence scores, all in a machine-readable structure.

**Command:**

```bash
cat target.xml
```

**Output:**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE nmaprun>
<?xml-stylesheet href="file:///usr/local/bin/../share/nmap/nmap.xsl" type="text/xsl"?>
<!-- Nmap 7.80 scan initiated Tue Jun 16 12:14:53 2020 as: nmap -p- -oA target 192.168.2.15 -->
<nmaprun scanner="nmap" args="nmap -p- -oA target 192.168.2.15" start="12145301719" startstr="Tue Jun 16 12:15:03 2020" version="7.80" xmloutputversion="1.04">
<scaninfo type="syn" protocol="tcp" numservices="65535" services="1-65535"/>
<verbose level="0"/>
<debugging level="0"/>
<host starttime="12145301719" endtime="12150323493"><status state="up" reason="arp-response" reason_ttl="0"/>
<address addr="192.168.2.15" addrtype="ipv4"/>
<address addr="DE:AD:00:00:BE:EF" addrtype="mac" vendor="Intel Corporate"/>
<hostnames>
</hostnames>
<ports><extraports state="closed" count="4">
<extrareasons reason="resets" count="4"/>
</extraports>
<port protocol="tcp" portid="22"><state state="open" reason="syn-ack" reason_ttl="64"/><service name="ssh" method="table" conf="3"/></port>
<port protocol="tcp" portid="25"><state state="open" reason="syn-ack" reason_ttl="64"/><service name="smtp" method="table" conf="3"/></port>
<port protocol="tcp" portid="80"><state state="open" reason="syn-ack" reason_ttl="64"/><service name="http" method="table" conf="3"/></port>
</ports>
<times srtt="52614" rttvar="75640" to="355174"/>
</host>
<runstats><finished time="12150323493" timestr="Tue Jun 16 12:14:53 2020" elapsed="10.22" summary="Nmap done at Tue Jun 16 12:15:03 2020; 1 IP address (1 host up) scanned in 10.22 seconds" exit="success"/><hosts up="1" down="0" total="1"/>
</runstats>
</nmaprun>
```

The XML file records the scan type (SYN), the protocol, the total number of services scanned, and the full port range. Each host entry contains the IP address, MAC address, vendor, and a separate block for every port that was found. Within each port block, the state, the reason it was assigned that state, the TTL, the service name, and the detection confidence level are all stored individually.

This format is what tools like Metasploit use when importing Nmap results. Any custom script or reporting tool that needs to process Nmap output programmatically will also expect XML. Because the structure is consistent and well-defined, parsing it reliably is straightforward regardless of how many hosts or ports were scanned.

---

## Converting XML to HTML

XML is not easy to read at a glance, which makes it less useful for sharing results with people who are not directly involved in the technical work. Nmap ships with an XSL stylesheet that can transform the XML output into a clean HTML report. The tool used for this conversion is xsltproc.

**Command:**

```bash
xsltproc target.xml -o target.html
```
**Output:**

<img width="2716" height="1284" alt="image" src="https://github.com/user-attachments/assets/316de29b-579d-4f31-8c32-5c78bbb51a74" />


Once the command runs, a target.html file is created in the same directory. Opening it in any browser produces a formatted, readable report with the scan results laid out in tables. This is particularly useful when preparing documentation for clients or stakeholders who need to understand the findings without reading raw scan data. It requires no additional setup beyond having xsltproc installed, which is available by default on most Linux distributions.
