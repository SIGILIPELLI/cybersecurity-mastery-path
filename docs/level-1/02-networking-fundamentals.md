# 02 · Networking Fundamentals for Security

You cannot secure — or attack, or investigate — what you don't understand.
Nearly every security control in this program sits somewhere on the network
stack, so this module builds the model you'll keep referencing: how data
actually moves from one machine to another, and where the practical security
issues live at each layer.

## 1. The OSI model, security-relevant version

The full seven-layer OSI model is taught in every networking course; here it
is filtered down to what actually matters for security work.

| Layer | Name | What it does | Security relevance |
|---|---|---|---|
| 7 | Application | HTTP, DNS, SMTP — what the user-facing protocol does | Most web app vulnerabilities (Module 6) live here |
| 4 | Transport | TCP/UDP — reliable vs. best-effort delivery, ports | Port scanning, firewalls, segmentation |
| 3 | Network | IP — addressing and routing between networks | IP spoofing, routing attacks, subnetting for segmentation |
| 2 | Data Link | MAC addresses, switches, local segment delivery | ARP spoofing, VLAN hopping |
| 1 | Physical | Cables, radio, physical medium | Wiretapping, rogue access points |

In practice, most of this course lives at layers 3, 4, and 7 — the layers you
can inspect with the tools in Module 8.

## 2. TCP/IP in practice

**IP (Internet Protocol)** gets a packet from one machine to another based on
an address. **TCP (Transmission Control Protocol)** sits on top of IP and adds
reliability: it establishes a connection, guarantees ordered delivery, and
retransmits lost data. **UDP (User Datagram Protocol)** skips all of that —
it's faster but offers no delivery guarantee, which is why DNS lookups and
video streaming often use it while file transfers and web pages use TCP.

### The TCP three-way handshake

Every TCP connection — including the one your browser makes to every website
— starts with this exchange:

```
Client                          Server
  |------ SYN (seq=x) --------->|      "I'd like to connect, my sequence starts at x"
  |<--- SYN-ACK (seq=y,ack=x+1)-|      "Acknowledged, my sequence starts at y"
  |------ ACK (ack=y+1) ------->|      "Acknowledged, connection established"
  |                             |
  |<====== data flows =========>|
```

This matters for security in two direct ways: a **SYN flood** attack (Level 2)
exploits this by sending SYN packets and never completing the handshake,
exhausting the server's connection table; and a **port scan** (Module 8) works
by sending SYN packets and reading what comes back — a SYN-ACK means the port
is open, nothing or a RST means it's closed or filtered.

## 3. Ports and common protocols

A port is a number (0–65535) that identifies which service on a machine a
connection is for. IP gets you to the right machine; the port gets you to the
right *application* on that machine.

| Port | Protocol | Purpose | Security note |
|---|---|---|---|
| 20/21 | FTP | File transfer | Sends credentials in plaintext — avoid |
| 22 | SSH | Encrypted remote shell | The standard secure replacement for Telnet |
| 23 | Telnet | Unencrypted remote shell | Never use on an untrusted network — plaintext everything |
| 25 | SMTP | Email sending | Source of spoofed-sender phishing (Module 7) |
| 53 | DNS | Name resolution | DNS spoofing/poisoning redirects users to malicious sites |
| 80 | HTTP | Unencrypted web traffic | Plaintext — anyone on the path can read it |
| 443 | HTTPS | Encrypted web traffic (TLS) | The standard for anything sensitive — see Module 4 |
| 445 | SMB | Windows file sharing | Frequent ransomware propagation vector (e.g., EternalBlue) |
| 3306 | MySQL | Database | Should never be exposed to the internet directly |
| 3389 | RDP | Windows Remote Desktop | Extremely common brute-force target when exposed |

!!! tip "The first question in any security review: what's listening?"
    A huge share of real breaches trace back to a service that was exposed to
    the internet and shouldn't have been — a database, an RDP port, an admin
    panel. Module 8 has you actually scan your own machine to see what's
    listening; get comfortable with the idea now that **every open port is
    attack surface**, whether or not it's currently vulnerable.

## 4. Packet basics — what's actually in a packet

A packet is layered like an envelope inside an envelope. A simplified TCP/IP
packet, outside-in:

```
+-------------------------------------------------------------+
| Ethernet header (src/dst MAC address)                        |
|  +---------------------------------------------------------+ |
|  | IP header (src/dst IP address, TTL, protocol)            | |
|  |  +-------------------------------------------------------+| |
|  |  | TCP/UDP header (src/dst port, sequence numbers, flags)| | |
|  |  |  +-----------------------------------------------------+ |
|  |  |  | Application data (HTTP request, DNS query, etc.)     | |
|  |  |  +-----------------------------------------------------+ |
|  |  +-------------------------------------------------------+| |
|  +---------------------------------------------------------+ |
+-------------------------------------------------------------+
```

Each layer strips its own header as the packet moves up the stack on the
receiving end. This is exactly what a packet capture tool like Wireshark
(Module 8) shows you: click a captured packet and you see each of these
layers as an expandable section, from Ethernet down to the raw application
payload.

## 5. Firewalls, NAT, and segmentation — a first look

A **firewall** is a rule-based filter that decides which traffic is allowed to
pass, typically based on source/destination IP, port, and protocol. The
default-deny principle — block everything, then explicitly allow only what's
needed — is the single most important firewall design decision, and Module 8
has you write real rules with `iptables`.

**NAT (Network Address Translation)** lets many private devices share one
public IP address (this is why your home devices all show the same external
IP). As a side effect, NAT provides a mild security benefit: an unsolicited
inbound connection has no NAT mapping to follow, so it's dropped by default —
though this is an accident of translation, not a real firewall, and should
never be relied on as your only protection.

**Network segmentation** splits a network into zones so that a compromise in
one zone doesn't automatically grant access to another — e.g., putting IoT
devices, guest Wi-Fi, and servers holding sensitive data on separate VLANs.
Level 2 Module 1 covers firewalls, IDS/IPS, and VPNs in depth; this section is
the vocabulary you need to get there.

## 6. DNS — the internet's phonebook, and its risks

**DNS (Domain Name System)** translates human-readable names (`example.com`)
into IP addresses. Because so much security depends on "am I really talking
to the site I think I am," DNS is a frequent attack target:

| Attack | What happens |
|---|---|
| **DNS spoofing/cache poisoning** | An attacker feeds a resolver a fake IP for a domain, redirecting victims to a malicious server |
| **Typosquatting** | Registering a similar-looking domain (`gοogle.com` with a Cyrillic o) to catch mistyped or visually-confused URLs |
| **DNS tunneling** | Smuggling data through DNS queries to bypass firewalls that don't inspect DNS traffic |

This is also why HTTPS (Module 4) matters even when DNS is compromised: a
valid TLS certificate is bound to a specific domain, so a spoofed DNS answer
pointing you at an attacker's server should trigger a certificate warning —
one of the reasons you should never click through a browser certificate
warning without understanding why it appeared.

## Key terms

| Term | Meaning |
|---|---|
| **Packet** | A unit of data with headers wrapping a payload, as it travels a network |
| **Port** | A number identifying which application on a host a connection targets |
| **Three-way handshake** | The SYN / SYN-ACK / ACK exchange that opens a TCP connection |
| **Firewall** | A rule-based filter deciding what traffic is allowed |
| **NAT** | Translating many private addresses to share one public address |
| **Segmentation** | Splitting a network into isolated zones to limit breach impact |
| **DNS** | The system translating domain names to IP addresses |
| **Attack surface** | Everything an attacker could potentially target or interact with |

## Exercise

You'll need a terminal (macOS/Linux built-in, or Windows with PowerShell/WSL).
No installation required for this one — Module 8 covers installing nmap and
Wireshark for deeper work.

1. **Trace a connection.** Run `ping example.com` and note the IP address it
   resolves to. Then run `nslookup example.com` (or `dig example.com` on
   macOS/Linux) and confirm the same IP appears in the DNS answer.

2. **Watch a real handshake and TLS negotiation from the outside.** Run
   `curl -v https://example.com` and read the verbose output. Identify: the
   IP it connected to, the port used, the TLS version negotiated, and the
   certificate's issuer.

3. **List your own open ports.** On macOS/Linux, run `netstat -an | grep LISTEN`
   (or `ss -tuln` on Linux); on Windows, run `netstat -an | findstr LISTENING`.
   For each listening port, identify the service (use the table in section 3
   as a reference) and write one sentence on whether it *should* be reachable
   from outside your machine.

4. **Written answer.** For one of the listening ports you found, describe: what
   protocol runs on it, what an attacker could try against it if it were
   exposed to the internet, and what layer of the OSI model (from section 1)
   the relevant attack would target.
