# 03 · Linux Security Hardening

Level 1 Module 3 covered OS security basics conceptually across platforms.
This module goes hands-on with **Linux server hardening** — the concrete
steps a defender takes to reduce a fresh Linux install's attack surface
before it goes anywhere near production, using a disposable VM or container
so you can break things safely.

!!! note "Lab setup"
    Use a disposable VM (VirtualBox/UTM/multipass Ubuntu Server image) or a
    Docker container for every command below. Never practice hardening
    commands — especially firewall and SSH changes — on a machine you can't
    afford to lock yourself out of.

## 1. The principle: minimize attack surface

Every open port, running service, installed package, and enabled user
account is something an attacker could potentially abuse. Hardening is
systematically removing everything that isn't needed:

```bash
# What's listening?
sudo ss -tulpn

# What's installed that you didn't explicitly ask for?
apt list --installed | wc -l   # Debian/Ubuntu
rpm -qa | wc -l                # RHEL/CentOS

# What services start on boot?
systemctl list-unit-files --state=enabled
```

## 2. SSH hardening

SSH is the most commonly attacked service on internet-facing Linux hosts.
Edit `/etc/ssh/sshd_config`:

```
# Disable direct root login -- forces attackers (and admins) through a
# named account first, which is auditable and can be MFA-protected
PermitRootLogin no

# Disable password auth entirely -- key-based auth resists brute force
PasswordAuthentication no
PubkeyAuthentication yes

# Change the default port -- doesn't stop a targeted attacker, but
# eliminates 95% of untargeted internet-wide scanning noise
Port 2222

# Limit which users/groups may connect over SSH at all
AllowUsers deploy admin

# Disconnect idle sessions
ClientAliveInterval 300
ClientAliveCountMax 2
```

```bash
sudo sshd -t              # validate config syntax before restarting
sudo systemctl restart sshd
```

Generate and deploy a key pair rather than relying on passwords:

```bash
ssh-keygen -t ed25519 -C "you@yourmachine"
ssh-copy-id -p 2222 deploy@yourhost
```

## 3. Firewall configuration (UFW)

```bash
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow 2222/tcp   # your new SSH port -- do this BEFORE enabling!
sudo ufw allow 443/tcp    # only the services you actually need
sudo ufw enable
sudo ufw status verbose
```

The order matters: allow your SSH port *before* enabling the firewall, or
you will lock yourself out of a remote box with no console access.

## 4. Automatic security updates

```bash
sudo apt install unattended-upgrades
sudo dpkg-reconfigure --priority=low unattended-upgrades
```

This applies security patches automatically without requiring a human to
remember — the single highest-leverage hardening step, since most breaches
exploit vulnerabilities that already had a patch available.

## 5. Least-privilege users and sudo

```bash
# Create a named, non-root admin account
sudo adduser deploy
sudo usermod -aG sudo deploy

# Audit who can sudo, and to what
sudo cat /etc/sudoers.d/*

# Never share accounts -- every human gets their own, for accountability
```

## 6. File permission and integrity basics

```bash
# Find world-writable files -- a classic privilege escalation vector
find / -xdev -type f -perm -0002 -ls 2>/dev/null

# Find SUID/SGID binaries -- run as the file owner, not the caller;
# unnecessary ones are a privilege-escalation path
find / -xdev \( -perm -4000 -o -perm -2000 \) -ls 2>/dev/null

# File integrity monitoring -- detect unauthorized changes to critical files
sudo apt install aide
sudo aideinit
sudo aide --check
```

## 7. Kernel and network hardening (`sysctl`)

```bash
# /etc/sysctl.d/99-hardening.conf
net.ipv4.conf.all.rp_filter = 1        # reject spoofed source addresses
net.ipv4.tcp_syncookies = 1            # mitigate SYN flood DoS
net.ipv4.conf.all.accept_redirects = 0 # ignore unsolicited ICMP redirects
net.ipv4.conf.all.log_martians = 1     # log packets with impossible addresses
```

```bash
sudo sysctl -p /etc/sysctl.d/99-hardening.conf
```

## 8. Mandatory access control (AppArmor / SELinux)

Discretionary permissions (`chmod`/`chown`) only restrict *who* can access a
file. Mandatory access control restricts *what a process is allowed to do
at all*, even as root, based on a policy — so a compromised web server
process can't read `/etc/shadow` even if the web server happens to be
running as root.

```bash
# Ubuntu/Debian ships AppArmor by default
sudo aa-status
sudo aa-enforce /etc/apparmor.d/usr.sbin.nginx

# RHEL/CentOS ships SELinux by default
getenforce
sudo setenforce 1
sestatus
```

## 9. A hardening checklist (CIS Benchmarks)

The **Center for Internet Security (CIS) Benchmarks** are the industry-
standard, freely available hardening checklists for every major OS and
platform. `lynis` automates auditing a box against these:

```bash
sudo apt install lynis
sudo lynis audit system
```

Lynis produces a scored report with specific, actionable findings —
run it before and after hardening to measure your progress.

## Key terms

| Term | Meaning |
|---|---|
| **Attack surface** | The sum of everything an attacker could potentially interact with or exploit |
| **Least privilege** | Every account/process gets only the access it strictly needs |
| **MAC (Mandatory Access Control)** | Kernel-enforced policy restricting process behavior beyond file permissions (AppArmor, SELinux) |
| **SUID/SGID** | File permission bits that run a binary as its owner/group rather than the caller — a common privilege-escalation vector if misused |
| **CIS Benchmark** | Industry-standard, vendor-neutral hardening checklist |
| **Unattended upgrades** | Automatic application of security patches without manual intervention |

## Exercise

1. Spin up a fresh Ubuntu Server VM or container.
2. Run `lynis audit system` and record the initial hardening score.
3. Apply sections 2–7 above, in order (SSH changes last, testing each
   change in a second terminal session before closing your first one).
4. Re-run `lynis audit system` and compare the score.
5. Find and document (don't remove yet) any SUID binaries on the box you
   don't recognize — research what each one is for before deciding whether
   it's actually needed.
