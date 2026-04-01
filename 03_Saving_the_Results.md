# Saving the Results

Every scan you run during a penetration test should be saved. Going back to earlier results lets you compare different scanning methods, track changes over time, and build accurate documentation for your final report. Nmap supports three output formats, each suited to a different purpose. You can save to any one of them individually or write all three at the same time.

The three formats are:

Normal output (-oN) saved with a .nmap file extension, grepable output (-oG) saved with a .gnmap file extension, and XML output (-oX) saved with a .xml file extension.

The option (-oA) writes all three formats simultaneously using a single filename prefix, which is the most practical choice during a real engagement.

---

## Saving All Formats at Once

**Command:**

```bash
sudo nmap 192.168.100.59 -p- -oA target
```

**Output:**

```
Starting Nmap 7.98 ( https://nmap.org ) at 2026-04-1 13:08 -0400
Nmap scan report for 192.168.100.59
Host is up (0.00073s latency).
Not shown: 65526 closed tcp ports (reset)
PORT     STATE SERVICE
22/tcp   open  ssh
80/tcp   open  http
139/tcp  open  netbios-ssn
143/tcp  open  imap
443/tcp  open  https
445/tcp  open  microsoft-ds
5001/tcp open  commplex-link
8080/tcp open  http-proxy
8081/tcp open  blackice-icecap
MAC Address: 08:00:27:00:14:D9 (Oracle VirtualBox virtual NIC)

Nmap done: 1 IP address (1 host up) scanned in 4.85 seconds

```

**Option Breakdown:**

| Option | Description |
|--------|-------------|
| 192.168.100.59 | The target being scanned |
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
# Nmap 7.98 scan initiated Wed Apr  1 13:08:40 2026 as: /usr/lib/nmap/nmap -p- -oA target 192.168.100.59
Nmap scan report for 192.168.100.59
Host is up (0.00073s latency).
Not shown: 65526 closed tcp ports (reset)
PORT     STATE SERVICE
22/tcp   open  ssh
80/tcp   open  http
139/tcp  open  netbios-ssn
143/tcp  open  imap
443/tcp  open  https
445/tcp  open  microsoft-ds
5001/tcp open  commplex-link
8080/tcp open  http-proxy
8081/tcp open  blackice-icecap
MAC Address: 08:00:27:00:14:D9 (Oracle VirtualBox virtual NIC)

# Nmap done at Wed Apr  1 13:08:45 2026 -- 1 IP address (1 host up) scanned in 4.85 seconds

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
# Nmap 7.98 scan initiated Wed Apr  1 13:08:40 2026 as: /usr/lib/nmap/nmap -p- -oA target 192.168.100.59
Host: 192.168.100.59 ()	Status: Up
Host: 192.168.100.59 ()	Ports: 22/open/tcp//ssh///, 80/open/tcp//http///, 139/open/tcp//netbios-ssn///, 143/open/tcp//imap///, 443/open/tcp//https///, 445/open/tcp//microsoft-ds///, 5001/open/tcp//commplex-link///, 8080/open/tcp//http-proxy///, 8081/open/tcp//blackice-icecap///	Ignored State: closed (65526)
# Nmap done at Wed Apr  1 13:08:45 2026 -- 1 IP address (1 host up) scanned in 4.85 seconds

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
<?xml-stylesheet href="file:///usr/share/nmap/nmap.xsl" type="text/xsl"?>
<!-- Nmap 7.98 scan initiated Wed Apr  1 13:08:40 2026 as: /usr/lib/nmap/nmap -p- -oA target 192.168.100.59 -->
<nmaprun scanner="nmap" args="/usr/lib/nmap/nmap -p- -oA target 192.168.100.59" start="1775063320" startstr="Wed Apr  1 13:08:40 2026" version="7.98" xmloutputversion="1.05">
<scaninfo type="syn" protocol="tcp" numservices="65535" services="1-65535"/>
<verbose level="0"/>
<debugging level="0"/>
<hosthint><status state="up" reason="arp-response" reason_ttl="0"/>
<address addr="192.168.100.59" addrtype="ipv4"/>
<address addr="08:00:27:00:14:D9" addrtype="mac" vendor="Oracle VirtualBox virtual NIC"/>
<hostnames>
</hostnames>
</hosthint>
<host starttime="1775063320" endtime="1775063325"><status state="up" reason="arp-response" reason_ttl="0"/>
<address addr="192.168.100.59" addrtype="ipv4"/>
<address addr="08:00:27:00:14:D9" addrtype="mac" vendor="Oracle VirtualBox virtual NIC"/>
<hostnames>
</hostnames>
<ports><extraports state="closed" count="65526">
<extrareasons reason="reset" count="65526" proto="tcp" ports="1-21,23-79,81-138,140-142,144-442,444,446-5000,5002-8079,8082-65535"/>
</extraports>
<port protocol="tcp" portid="22"><state state="open" reason="syn-ack" reason_ttl="64"/><service name="ssh" method="table" conf="3"/></port>
<port protocol="tcp" portid="80"><state state="open" reason="syn-ack" reason_ttl="64"/><service name="http" method="table" conf="3"/></port>
<port protocol="tcp" portid="139"><state state="open" reason="syn-ack" reason_ttl="64"/><service name="netbios-ssn" method="table" conf="3"/></port>
<port protocol="tcp" portid="143"><state state="open" reason="syn-ack" reason_ttl="64"/><service name="imap" method="table" conf="3"/></port>
<port protocol="tcp" portid="443"><state state="open" reason="syn-ack" reason_ttl="64"/><service name="https" method="table" conf="3"/></port>
<port protocol="tcp" portid="445"><state state="open" reason="syn-ack" reason_ttl="64"/><service name="microsoft-ds" method="table" conf="3"/></port>
<port protocol="tcp" portid="5001"><state state="open" reason="syn-ack" reason_ttl="64"/><service name="commplex-link" method="table" conf="3"/></port>
<port protocol="tcp" portid="8080"><state state="open" reason="syn-ack" reason_ttl="64"/><service name="http-proxy" method="table" conf="3"/></port>
<port protocol="tcp" portid="8081"><state state="open" reason="syn-ack" reason_ttl="64"/><service name="blackice-icecap" method="table" conf="3"/></port>
</ports>
<times srtt="728" rttvar="380" to="100000"/>
</host>
<runstats><finished time="1775063325" timestr="Wed Apr  1 13:08:45 2026" summary="Nmap done at Wed Apr  1 13:08:45 2026; 1 IP address (1 host up) scanned in 4.85 seconds" elapsed="4.85" exit="success"/><hosts up="1" down="0" total="1"/>
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

<img width="1537" height="765" alt="image" src="https://github.com/user-attachments/assets/a0dfeb8f-4731-4b1b-9fa5-0464981f866a" />



Once the command runs, a target.html file is created in the same directory. Opening it in any browser produces a formatted, readable report with the scan results laid out in tables. This is particularly useful when preparing documentation for clients or stakeholders who need to understand the findings without reading raw scan data. It requires no additional setup beyond having xsltproc installed, which is available by default on most Linux distributions.
