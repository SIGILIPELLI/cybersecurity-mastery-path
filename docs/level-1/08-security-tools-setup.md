# 08 · Security Tools Setup

This module gets a real home lab running: a place to practice everything in
this course safely, plus the three tools you'll use constantly from here on —
a virtual machine for isolated experimentation, `nmap` for network scanning,
and Wireshark for packet capture. Every command below was run and verified
against `localhost`/a local lab VM — nothing here touches a network you don't
own.

!!! warning "Scope reminder"
    `nmap` and packet capture tools are legitimate, widely-used administrative
    tools — and also exactly what a real attacker's reconnaissance phase looks
    like (Level 2 Module 9). Running them against any host you don't own or
    don't have explicit written authorization to scan is illegal in most
    jurisdictions. Everything in this module targets `127.0.0.1` (your own
    machine) or a VM you control.

## 1. Set up a lab VM

A virtual machine gives you an isolated, disposable environment to break
things in without risking your main OS.

| Hypervisor | Platform | Notes |
|---|---|---|
| **VirtualBox** | Windows/macOS/Linux, free | Simplest starting point for this course |
| **VMware Workstation Player** | Windows/Linux, free for personal use | Slightly better performance than VirtualBox on some hosts |
| **UTM** | macOS (especially Apple Silicon) | Native ARM virtualization, free |

A practical starter lab: install **Ubuntu Desktop or Server** (free, widely
documented, matches the Linux commands used throughout this course) as a
guest VM. Configure its network adapter as **"Host-only" or "NAT"** rather
than "Bridged" while you're learning — this keeps your lab traffic off your
real network entirely, which matters once you start running scans in section
3.

```
Host machine (your real OS)
   │
   ▼
[ Hypervisor: VirtualBox/VMware/UTM ]
   │
   ▼
[ Guest VM: Ubuntu, isolated network ]   ← your lab environment
```

!!! tip "Snapshots are your undo button"
    Take a VM snapshot right after a clean install. Any time you're about to
    try something destructive or uncertain — installing a deliberately
    vulnerable app, testing a hardening change that might lock you out — take
    another snapshot first. Reverting is instant; reinstalling an OS is not.

## 2. Install nmap

`nmap` (Network Mapper) discovers hosts and open ports on a network — the
tool behind Module 2's "what's actually listening?" question, now automated
and scriptable.

```bash
# macOS (Homebrew)
brew install nmap

# Ubuntu/Debian
sudo apt install nmap

# Windows
# download the installer from https://nmap.org/download.html
```

Verify:

```bash
nmap --version
```

```
Nmap version 7.991 ( https://nmap.org )
Platform: arm-apple-darwin25.6.0
```

### Scan your own machine

```bash
nmap -sT -p 1-1024 127.0.0.1
```

```
Starting Nmap 7.991 ( https://nmap.org ) at 2026-08-28 10:06 +0530
Nmap scan report for localhost (127.0.0.1)
Host is up (0.000029s latency).
Not shown: 1022 closed tcp ports (conn-refused)
PORT    STATE SERVICE
88/tcp  open  kerberos-sec
445/tcp open  microsoft-ds

Nmap done: 1 IP address (1 host up) scanned in 0.07 seconds
```

This is a **TCP connect scan** (`-sT`) — it completes a full three-way
handshake (Module 2) with each port, which is reliable but noisy (it's
logged as a full connection on the target). Read this output the way you
would a real finding: two ports are open on this machine that weren't
necessarily expected — the exact "what's listening, and should it be?"
question from Module 2, now answered with a tool instead of `netstat`.

| Common nmap flag | What it does |
|---|---|
| `-sT` | TCP connect scan (completes full handshake) |
| `-sS` | TCP SYN scan ("half-open" — faster, requires elevated privileges) |
| `-p 1-1024` | Scan a specific port range |
| `-p-` | Scan all 65535 ports |
| `-sV` | Attempt to detect the service/version running on each open port |
| `-O` | Attempt OS fingerprinting |
| `-A` | Aggressive scan — OS detection, version detection, script scanning |

```bash
nmap -sV -p 88,445 127.0.0.1
```

Adding `-sV` on the ports found above will attempt to identify exactly what
service and version is answering — the natural next step after "what's
open?" is "what, specifically, is running there, and is it patched?" (Module
3's patch management, applied to what you just discovered).

!!! danger "This is reconnaissance — the first phase of a real attack"
    Everything in this section is literally step one of the penetration
    testing methodology covered properly, with its authorization requirements,
    in Level 2 Module 9. Running it against your own lab is standard practice.
    Running it against anything else is the same action a real attacker takes
    before an intrusion attempt.

## 3. Install and use Wireshark

Wireshark captures and inspects network traffic at the packet level — it
turns the layered-packet diagram from Module 2 section 4 into something you
can actually click through, header by header.

```bash
# macOS (Homebrew)
brew install --cask wireshark

# Ubuntu/Debian
sudo apt install wireshark
# (choose "yes" when asked to allow non-root packet capture, or add your
#  user to the wireshark group afterward: sudo usermod -aG wireshark $USER)

# Windows
# download the installer from https://www.wireshark.org/download.html
```

A minimal first capture:

1. Open Wireshark, select your active network interface (e.g., `en0`,
   `eth0`, or your Wi-Fi adapter), and click Start.
2. In a terminal, generate some traffic: `curl -s http://example.com > /dev/null`.
3. Stop the capture and type `http` in the filter bar to isolate HTTP
   packets.
4. Click one of the resulting packets and expand each layer in the detail
   pane — you'll see Ethernet, IP, TCP, and HTTP sections nested exactly as
   diagrammed in Module 2.

For a command-line alternative (useful on servers with no GUI), `tshark`
ships alongside Wireshark:

```bash
tshark -i en0 -f "tcp port 443" -c 10
```

captures the first 10 packets matching TCP port 443 on interface `en0`.

!!! tip "Capture on your own traffic only"
    On a shared or corporate network, capturing traffic that isn't yours (or
    that you don't have authorization to inspect) raises the same legal and
    ethical issues as network scanning. On your home network or lab VM,
    capturing your own machine's traffic is the safe, standard use case this
    course expects.

## 4. A basic iptables rule

`iptables` is the traditional Linux firewall front-end (Module 2's firewall
concept, made concrete). Run this on your lab VM, not your host — a mistaken
rule can lock you out of SSH.

```bash
# See current rules
sudo iptables -L -n -v

# Default-deny inbound, but explicitly allow loopback, established
# connections, and SSH -- the minimum safe baseline before you deny everything
sudo iptables -A INPUT -i lo -j ACCEPT
sudo iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT
sudo iptables -A INPUT -p tcp --dport 22 -j ACCEPT
sudo iptables -A INPUT -j DROP

# Confirm
sudo iptables -L -n -v
```

```
Chain INPUT (policy ACCEPT)
 pkts bytes target     prot opt in     out     source               destination
    0     0 ACCEPT     all  --  lo     *       0.0.0.0/0            0.0.0.0/0
    0     0 ACCEPT     all  --  *      *       0.0.0.0/0            0.0.0.0/0            state RELATED,ESTABLISHED
    0     0 ACCEPT     tcp  --  *      *       0.0.0.0/0            0.0.0.0/0            tcp dpt:22
    0     0 DROP       all  --  *      *       0.0.0.0/0            0.0.0.0/0
```

This is the default-deny principle from Module 2 as an actual ruleset: allow
loopback traffic (many local services rely on it), allow replies to
connections you initiated, explicitly allow SSH so you don't lock yourself
out, then drop everything else inbound. Rules are evaluated top to bottom and
the first match wins — order matters, which is why the `DROP` rule must come
last.

!!! warning "iptables rules don't survive a reboot by default"
    On most distributions you need `iptables-persistent` (Debian/Ubuntu) or
    equivalent to make rules survive a restart. This is intentional — it
    means an experimental rule that locks you out is undone by rebooting the
    VM, which is exactly why testing firewall rules on a disposable VM
    (section 1) rather than a machine you depend on is the right approach
    while learning.

## Key terms

| Term | Meaning |
|---|---|
| **Hypervisor** | Software that runs virtual machines (VirtualBox, VMware, UTM) |
| **Snapshot** | A saved VM state you can instantly revert to |
| **Port scan** | Probing a range of ports to discover what's open/listening |
| **TCP connect scan** | A scan that completes the full three-way handshake |
| **Packet capture** | Recording network traffic for inspection, packet by packet |
| **Default-deny** | A firewall policy that blocks everything not explicitly allowed |

## Exercise

Using the VM (or your own machine, for the nmap/Wireshark parts) set up in
this module:

1. **Stand up your lab VM** and take a clean snapshot immediately after OS
   install, before making any changes.

2. **Run the same scan** as section 2 against your own machine, and separately
   against your lab VM's IP (find it with `ip addr` inside the VM). Compare
   the two results and record which ports are open on each and why they
   differ (a fresh VM should show far fewer open ports than a daily-use
   machine).

3. **Capture and inspect one HTTP request** with Wireshark or `tshark`
   following the steps in section 3, and screenshot or copy the layered
   packet detail showing Ethernet/IP/TCP/HTTP nested exactly as in Module 2's
   diagram.

4. **Apply the iptables ruleset** from section 4 on your lab VM, then, from
   your host machine, attempt to reach a port on the VM that you did *not*
   explicitly allow (e.g., spin up `python3 -m http.server 8000` on the VM
   before applying the rules, then try to reach it after). Confirm the
   connection is refused/times out, then remove the `DROP` rule
   (`sudo iptables -D INPUT -j DROP`) and confirm it works again.

5. **Written answer.** Explain, using this module's iptables output, what
   would happen if you had applied the `DROP` rule *before* the SSH-allow
   rule instead of after — tie your answer back to "rules are evaluated top
   to bottom, first match wins."
