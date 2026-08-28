# 03 · Operating System Security Basics

The operating system is where every security control eventually gets
enforced — a firewall rule, a file permission, a login prompt are all OS
mechanisms. This module covers the concepts common to any OS (users,
permissions, the principle of least privilege) and the concrete hardening
steps for the two you'll actually touch: Linux and Windows.

## 1. Users, groups, and the principle of least privilege

Every mainstream OS is built around the idea that not every user — or every
*process* — should be able to do everything. Two closely related concepts:

- **Users** identify who (or what service account) is acting.
- **Groups** collect users so permissions can be assigned once, to the group,
  rather than repeated per user.

**The principle of least privilege (PoLP)**: every user, program, and process
should have the *minimum* access required to do its job, and nothing more.
This single idea underlies almost every OS and application security decision
in this course, and gets a full module of its own at the access-control level
(Module 5) — this module is where it starts at the OS layer.

| Bad practice | Why it violates least privilege | Better practice |
|---|---|---|
| Everyone logs in as root/Administrator daily | Any mistake or compromised process has full system control | Use a standard account; elevate only when needed (`sudo`, UAC) |
| A web server process runs as root | A vulnerability in the web app becomes a full system compromise | Run it as a dedicated, unprivileged service account |
| One shared admin password for the whole team | No accountability, can't revoke access for one person | Individual accounts, each granted only what's needed |

## 2. Linux permissions

Every file and directory in Linux has an owner, a group, and permission bits
for three categories: the **owner**, the **group**, and **others**.

```bash
$ ls -l secrets.txt
-rw-r----- 1 alice devteam 128 Aug 20 10:00 secrets.txt
```

Reading this left to right: `-` (regular file), then three permission
triplets — `rw-` for the owner (alice: read+write), `r--` for the group
(devteam: read only), `---` for everyone else (no access at all).

| Symbol | Meaning on a file | Meaning on a directory |
|---|---|---|
| `r` | Read the file's contents | List the directory's contents |
| `w` | Modify the file | Create/delete files inside it |
| `x` | Execute the file as a program/script | Enter the directory (`cd` into it) |

Permissions are also expressed as octal numbers — one digit per triplet,
summing `r=4, w=2, x=1`:

```bash
chmod 640 secrets.txt   # rw-r-----  : owner read/write, group read, others nothing
chmod 755 script.sh     # rwxr-xr-x : owner full, everyone else read+execute
chmod 600 id_rsa        # rw-------  : owner only -- the standard for private keys
```

!!! warning "`chmod 777` is almost never the right answer"
    `777` means read/write/execute for *everyone* on the system. It's the most
    common "just make the permissions error go away" fix beginners reach for,
    and it is very rarely correct — it turns a file-permission bug into a
    security hole. If a permission error appears, find out what owner/group
    the process actually needs, not the widest possible grant.

`sudo` lets an authorized user run a specific command as another user
(usually root) without logging in as root. `/etc/sudoers` (edited with
`visudo`, never a plain text editor, to avoid syntax errors locking you out)
controls exactly who can run what as whom — the mechanical enforcement of
least privilege for admin tasks.

## 3. Windows permissions and UAC

Windows uses **NTFS permissions**, conceptually similar to Linux but more
granular — instead of three fixed categories, an **Access Control List
(ACL)** attaches a list of specific users/groups, each with their own
permission set (Read, Write, Modify, Full Control, and more).

**User Account Control (UAC)** is Windows' least-privilege enforcement: even
an administrator account runs day-to-day with a *standard* token, and any
action requiring elevated rights triggers the familiar "Do you want to allow
this app to make changes?" prompt. Disabling UAC (a real setting some users
turn off out of prompt fatigue) removes this safety net entirely — any
program you run, including malware, silently gets full admin rights.

| Concept | Linux | Windows |
|---|---|---|
| Superuser | `root` | `Administrator` (built-in, usually disabled by default) |
| Temporary elevation | `sudo` | UAC prompt |
| Permission model | Owner/Group/Other + bits | ACL — arbitrary list of user/permission pairs |
| Where accounts live | `/etc/passwd`, LDAP | Local SAM database, or Active Directory (Level 2 Module 4) |

## 4. Patch management

Most real-world breaches don't use a novel technique — they use a known
vulnerability that already had a patch available, against a system that
hadn't installed it. Patch management is the practice of tracking and
applying security updates promptly.

```bash
# Debian/Ubuntu -- check and apply updates
sudo apt update && sudo apt list --upgradable
sudo apt upgrade -y

# Fedora/RHEL
sudo dnf check-update
sudo dnf upgrade -y

# macOS
softwareupdate --list
softwareupdate --install --all
```

```powershell
# Windows -- check for updates from PowerShell
Get-WindowsUpdate          # requires the PSWindowsUpdate module
# or simply: Settings > Windows Update > Check for updates
```

!!! note "Why 'we'll patch it next quarter' is a real risk decision"
    The time between a vulnerability becoming public and attackers actively
    exploiting it in the wild is often measured in days, not months —
    especially once a proof-of-concept is published. Delaying a patch is a
    genuine, quantifiable risk trade-off (recall Module 1's risk formula), not
    an administrative inconvenience — which is why patch cadence is often a
    compliance requirement (Level 3 Module 9), not just good practice.

## 5. Hardening baselines

"Hardening" means reducing a system's attack surface by turning off what you
don't need and locking down what you do. A short, concrete starting checklist
for a freshly installed machine:

**Linux:**

- [ ] Disable root SSH login (`PermitRootLogin no` in `/etc/ssh/sshd_config`)
- [ ] Disable password SSH auth in favor of key-based auth
      (`PasswordAuthentication no`)
- [ ] Remove or disable unused services (`systemctl list-unit-files --state=enabled`)
- [ ] Enable a firewall with default-deny inbound (`ufw enable`, or `iptables`
      — Module 8)
- [ ] Keep the system on automatic security updates where feasible
- [ ] Ensure no service runs as root unless it genuinely requires it

**Windows:**

- [ ] Keep UAC enabled
- [ ] Disable the built-in Administrator account, use a named admin account
      instead (auditability)
- [ ] Enable Windows Defender / an endpoint protection product, and keep
      definitions current
- [ ] Enable Windows Firewall with default-deny inbound
- [ ] Turn on BitLocker (disk encryption) on laptops
- [ ] Disable unused services and legacy protocols (e.g., SMBv1)

These are the same categories a professional hardening standard like CIS
Benchmarks covers in far more depth — this list is the 80/20 version you'll
actually apply on your own lab machine in Module 8 and audit in Module 10's
project.

## Key terms

| Term | Meaning |
|---|---|
| **Least privilege** | Grant the minimum access needed, no more |
| **ACL** | Access Control List — a list of user/permission pairs on a resource |
| **UAC** | User Account Control — Windows' privilege-elevation prompt |
| **Hardening** | Reducing attack surface by disabling/restricting what isn't needed |
| **Patch management** | The process of tracking and applying security updates |
| **Privilege escalation** | Gaining higher access than originally granted (Level 3 Module 1) |

## Exercise

On a machine you own (a personal Linux VM is ideal — Module 8 sets one up if
you don't have one yet):

1. **Inspect real permissions.** Run `ls -l /etc/shadow` (Linux) and explain,
   in writing, why this file is readable only by root even though `/etc/passwd`
   (which lists usernames) is world-readable. What would go wrong if
   `/etc/shadow` were world-readable?

2. **Practice safe permission changes.** Create a file, set it to `644`, then
   change it to `600` with `chmod`. Explain in one sentence why SSH refuses to
   use a private key file that has group- or world-read permissions.

3. **Audit your own account.** On whatever machine you're using right now,
   determine: are you running as an admin/root account day-to-day, or a
   standard account that elevates when needed? If you're on an always-admin
   account, this is your first real hardening action — document the change
   you'd make (you don't have to make it immediately if it would disrupt your
   main machine).

4. **Patch check.** Run the patch-check command from section 4 appropriate to
   your OS and record how many updates are pending, and whether any are
   flagged as security-related.
