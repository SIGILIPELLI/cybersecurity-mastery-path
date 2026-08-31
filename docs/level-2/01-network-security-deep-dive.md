# 01 · Network Security Deep Dive

Level 1 Module 2 covered how packets move. This module goes one layer
deeper: how firewalls actually decide what to drop, how an IDS spots an
attack in traffic, and how to read a packet capture yourself instead of
trusting a summary. You'll run a local firewall, generate traffic that
should and shouldn't pass, and inspect it in Wireshark.

!!! warning "Scope: your own machine and lab VMs only"
    Every capture and scan in this module runs against `127.0.0.1` or a
    local VM you control. Capturing or scanning traffic on a network you
    don't own or have written authorization to test is illegal in most
    jurisdictions.

## 1. Stateful vs. stateless firewalls

A **stateless** (packet-filtering) firewall evaluates each packet in
isolation against a rule list — source/destination IP, port, protocol. A
**stateful** firewall tracks connections: it remembers that `10.0.0.5`
opened an outbound TCP connection to port 443, and automatically allows the
return traffic for that specific connection without a separate inbound
rule. Nearly every modern firewall (iptables/nftables, Windows Firewall,
cloud security groups) is stateful by default — this is what lets "allow
outbound, deny inbound" work as a sane default policy.

| Type | Decision basis | Example |
|---|---|---|
| Stateless | Each packet alone | Classic ACLs on old routers |
| Stateful | Connection state table | iptables `-m state`, AWS Security Groups |
| Application-layer (NGFW/WAF) | Payload content, not just headers | Blocks a SQLi pattern even on an "allowed" port 443 |

## 2. Build and test a local firewall policy

On a Linux VM (or WSL), inspect and set a default-deny inbound policy with
`nftables`:

```bash
sudo nft list ruleset
```

```bash
sudo nft add table inet filter
sudo nft add chain inet filter input { type filter hook input priority 0 \; policy drop \; }
sudo nft add rule inet filter input ct state established,related accept
sudo nft add rule inet filter input iif lo accept
sudo nft add rule inet filter input tcp dport 22 accept
```

This reads as: drop everything inbound by default, except traffic that's
part of an already-established connection (`ct state established,related`
— the stateful part), loopback, and new SSH connections on port 22.

Test it from another host or namespace:

```bash
nc -zv <vm-ip> 22   # succeeds — explicitly allowed
nc -zv <vm-ip> 80   # times out — no rule allows it, default policy drops it
```

## 3. Capture and read the traffic

Start a capture while you run the test above:

```bash
sudo tcpdump -i any -w /tmp/fw_test.pcap port 22 or port 80
```

Open `/tmp/fw_test.pcap` in Wireshark (or `tcpdump -r /tmp/fw_test.pcap -nn`
on the CLI) and find the three-way handshake for the port 22 attempt:

```
SYN      10.0.0.10 -> 10.0.0.5:22
SYN,ACK  10.0.0.5 -> 10.0.0.10:22
ACK      10.0.0.10 -> 10.0.0.5:22
```

For the port 80 attempt you'll see only the outgoing `SYN` — no response at
all, because `policy drop` silently discards the packet rather than
replying with a `RST` (a `REJECT` rule would instead send a visible
`RST`/ICMP unreachable). This silent-vs-rejected distinction matters for
recon: an attacker running a port scan can often tell a "closed but
responsive" port from a "filtered/dropped" port, which leaks information
about which policy is in effect.

## 4. Detecting a scan with an IDS

Install Suricata (or use Zeek) on the same VM and point it at the interface:

```bash
sudo apt install suricata
sudo suricata -i eth0 -c /etc/suricata/suricata.yaml
```

Run a scan against the VM from another host:

```bash
nmap -sS -p 1-1000 10.0.0.5
```

Tail the alert log:

```bash
tail -f /var/log/suricata/fast.log
```

```
08/29/2026-10:14:02  [**] [1:2210036:1] SURICATA STREAM Packet with invalid ack [**]
08/29/2026-10:14:03  [**] [1:2001219:19] ET SCAN Potential SSH Scan [**] {TCP} 10.0.0.10:51321 -> 10.0.0.5:22
```

Suricata matched the burst of half-open connections to many ports in a
short window against a signature (`ET SCAN`) — this is the same principle
behind every network IDS: known-bad *patterns* in traffic, whether that's a
scan signature, a known exploit byte sequence, or a beaconing interval to a
known-bad IP.

## 5. Segmentation and the principle of least network access

The other half of network security is architectural, not per-packet: **network
segmentation**. A flat network where every host can reach every other host
means a single compromised laptop can pivot to the database server. VLANs,
subnets with firewall rules between them, and a DMZ for internet-facing
services all implement the same idea as Level 1's least privilege, applied
to network paths instead of user permissions — a web server should be able
to reach *only* its database on *only* the DB port, nothing else.

## Key terms

| Term | Meaning |
|---|---|
| **Stateful firewall** | Tracks connection state to auto-allow return traffic |
| **Default-deny** | Block everything not explicitly allowed |
| **DROP vs. REJECT** | Silent discard vs. an explicit refusal response |
| **IDS/IPS** | Intrusion Detection/Prevention System — flags (or blocks) malicious traffic patterns |
| **Segmentation** | Splitting a network into zones with controlled paths between them |
| **DMZ** | A network zone for internet-facing hosts, isolated from the internal network |

## Exercise

1. Reproduce the `nftables` policy above in a VM and confirm port 22 is
   reachable and port 80 is not, capturing both attempts with `tcpdump`.
2. Change the port 80 rule's policy from `drop` to an explicit `reject`
   (`nft insert rule inet filter input tcp dport 80 reject with tcp reset`)
   and capture the difference in the response packet.
3. Install Suricata (or Zeek) in your lab VM, run an `nmap -sS` scan against
   it from another VM, and paste the resulting alert log line.
4. Design (on paper — a diagram is fine) a three-zone segmentation for a
   small company: a public web server, an internal database, and employee
   workstations. List exactly which zone can initiate a connection to
   which, and on what ports.
5. Written answer: explain why "default-deny inbound, default-allow
   outbound with logging" is a common baseline policy, and one scenario
   where you'd also want to restrict outbound traffic.
