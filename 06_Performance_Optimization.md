# Performance Optimization

Scan performance becomes a critical factor when working across large networks or operating under constrained bandwidth conditions. Nmap provides several options to control how aggressively it scans: the timing template (`-T <0-5>`), packet frequency (`--min-parallelism`), round-trip timeouts (`--max-rtt-timeout`), simultaneous packet rate (`--min-rate`), and retry attempts (`--max-retries`). Understanding these options allows you to balance speed against accuracy depending on the engagement type.

---

## Timeouts

Every packet Nmap sends requires a response, and the time between sending a packet and receiving its reply is called the Round-Trip Time (RTT). By default, Nmap begins with an RTT timeout of 100ms and adjusts dynamically. Setting this value too low causes Nmap to give up on hosts before they have had a chance to respond, which leads to missed results.

The example below scans a /24 subnet across the top 100 ports, first with default settings, then with a reduced RTT timeout.

**Default Scan**

```bash
sudo nmap 192.168.100.0/24 -F
```

```
Nmap done: 256 IP addresses (10 hosts up) scanned in 39.44 seconds
```

**Optimized RTT**

```bash
sudo nmap 192.168.100.0/24 -F --initial-rtt-timeout 50ms --max-rtt-timeout 100ms
```

```
Nmap done: 256 IP addresses (8 hosts up) scanned in 12.29 seconds
```

| Option | Description |
|---|---|
| `192.168.100.0/24` | Target network range |
| `-F` | Scan top 100 ports |
| `--initial-rtt-timeout 50ms` | Sets the starting RTT timeout value |
| `--max-rtt-timeout 100ms` | Sets the upper RTT timeout limit |

The optimized scan completed in roughly a quarter of the time, but two hosts were missed entirely. This demonstrates that reducing RTT timeouts too aggressively introduces the risk of incomplete results, a tradeoff that needs to be considered carefully based on the network environment.

---

## Max Retries

When Nmap does not receive a response for a port, it retries by default up to 10 times before moving on. Reducing this value with `--max-retries` speeds up the scan but increases the chance of missing open ports on slower or less responsive hosts.

**Default Scan**

```bash
sudo nmap 192.168.100.0/24 -F | grep "/tcp" | wc -l
```

```
23
```

**Reduced Retries**

```bash
sudo nmap 192.168.100.0/24 -F --max-retries 0 | grep "/tcp" | wc -l
```

```
21
```

| Option | Description |
|---|---|
| `192.168.100.0/24` | Target network range |
| `-F` | Scan top 100 ports |
| `--max-retries 0` | Disables retries entirely |

Two ports were missed with retries set to zero. As with RTT tuning, speed gains come at the cost of reliability.

---

## Rates

In white-box engagements where your scanner has been whitelisted by security controls, you can push scan speed significantly by setting a minimum packet rate with `--min-rate`. This tells Nmap to send at least the specified number of packets per second, which is particularly effective on high-bandwidth networks.

**Default Scan**

```bash
sudo nmap 192.168.100.0/24 -F -oN tnet.default
```

```
Nmap done: 256 IP addresses (10 hosts up) scanned in 29.83 seconds
```

**Optimized Scan**

```bash
sudo nmap 192.168.100.0/24 -F -oN tnet.minrate300 --min-rate 300
```

```
Nmap done: 256 IP addresses (10 hosts up) scanned in 8.67 seconds
```

| Option | Description |
|---|---|
| `192.168.100.0/24` | Target network range |
| `-F` | Scan top 100 ports |
| `-oN tnet.minrate300` | Saves output to a file in normal format |
| `--min-rate 300` | Sends a minimum of 300 packets per second |

**Comparing open port counts:**

```bash
cat tnet.default | grep "/tcp" | wc -l
23

cat tnet.minrate300 | grep "/tcp" | wc -l
23
```

Both scans returned identical results, with the optimized scan finishing in under a third of the time. When network conditions allow, setting a minimum rate is one of the most effective ways to reduce scan duration without sacrificing accuracy.

---

## Timing Templates

Manual tuning is not always practical, especially during black-box assessments where network characteristics are unknown. For these situations, Nmap provides six predefined timing templates that bundle sensible defaults together under a single flag.

| Template | Flag |
|---|---|
| Paranoid | `-T0` |
| Sneaky | `-T1` |
| Polite | `-T2` |
| Normal | `-T3` |
| Aggressive | `-T4` |
| Insane | `-T5` |

The default template when nothing is specified is Normal (`-T3`). Higher values increase scan aggressiveness, which reduces scan time but also increases the likelihood of triggering IDS alerts or overwhelming unstable hosts. Lower values slow the scan down to reduce noise on the network.

**Default Scan**

```bash
sudo nmap 192.168.100.0/24 -F -oN tnet.default
```

```
Nmap done: 256 IP addresses (10 hosts up) scanned in 32.44 seconds
```

**Insane Scan**

```bash
sudo nmap 192.168.100.0/24 -F -oN tnet.T5 -T 5
```

```
Nmap done: 256 IP addresses (10 hosts up) scanned in 18.07 seconds
```

| Option | Description |
|---|---|
| `192.168.100.0/24` | Target network range |
| `-F` | Scan top 100 ports |
| `-oN tnet.T5` | Saves output to a file in normal format |
| `-T5` | Applies the insane timing template |

**Comparing open port counts:**

```bash
cat tnet.default | grep "/tcp" | wc -l
23

cat tnet.T5 | grep "/tcp" | wc -l
23
```

In this case both templates found the same ports, but results can diverge on less stable networks. The full breakdown of what each timing template sets internally is documented at [nmap.org/book/performance-timing-templates.html](https://nmap.org/book/performance-timing-templates.html).

---

The key takeaway across all of these options is that faster is not always better. Every performance optimization involves a tradeoff with result completeness. In time-sensitive engagements, higher speeds are acceptable when the network is stable and the scope is well-defined. In more sensitive assessments, slower and more thorough settings are the safer choice.
