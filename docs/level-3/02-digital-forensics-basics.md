# 02 · Digital Forensics Basics

Incident response (Level 1 Module 9) tells you *what to do* when something
goes wrong. Digital forensics is the discipline of proving *what actually
happened*, in a way that survives scrutiny — whether that scrutiny comes
from a courtroom, a regulator, or your own post-incident review board.

!!! warning "Authorized systems and lab images only"
    Practice on disk/memory images you create yourself (a spare VM, a
    downloaded forensics training image such as those from
    DFIR.training or CFReDS), never on a system you do not own or have
    explicit written authorization to examine.

## 1. The forensic mindset: chain of custody and evidence integrity

Forensics is different from ordinary troubleshooting because every action
must be **defensible**. Two ideas underpin everything else:

- **Chain of custody** — an unbroken, documented record of who touched
  the evidence, when, and why. A gap in the chain can get evidence thrown
  out entirely.
- **Integrity via hashing** — before analysis, hash the original evidence.
  Every subsequent step works on a *copy*, and you re-hash to prove
  nothing changed.

```bash
# Hash the original evidence immediately, before touching it further
sha256sum disk.img > disk.img.sha256

# Work only on a copy; verify the copy matches
cp disk.img disk_working_copy.img
sha256sum -c disk.img.sha256
```

A basic evidence log records: item description, source, date/time
acquired, acquiring analyst, hash value, and every hand-off after that.

## 2. Order of volatility

RFC 3227 defines an order for collecting evidence, most volatile first —
because some evidence disappears the moment you power something off:

```
1. CPU registers, cache
2. RAM (running processes, network connections, encryption keys)
3. Network state (active connections, ARP cache, routing table)
4. Running processes
5. Disk
6. Remote logging / monitoring data
7. Physical configuration, network topology
8. Archival media / backups
```

This is why the first IR instinct of "just unplug it" is often wrong —
pulling power on a live-compromise host destroys RAM evidence (in-memory
malware, decrypted data, active C2 sessions) that disk imaging alone will
never recover.

## 3. Memory acquisition and analysis

```bash
# Acquire memory from a live Linux host (run from a trusted USB, not the
# host's own binaries, to avoid rootkit-tampered tools)
sudo ./avml memory.lime

# Windows equivalent
winpmem_mini_x64.exe memory.raw
```

Analyze with the Volatility Framework:

```bash
# Identify the OS profile
vol.py -f memory.raw imageinfo

# List running processes -- look for anything unexpected or spoofing a
# legitimate name (svch0st.exe vs svchost.exe)
vol.py -f memory.raw --profile=Win10x64 pslist

# Network connections at time of capture
vol.py -f memory.raw --profile=Win10x64 netscan

# Dump a suspicious process for further analysis
vol.py -f memory.raw --profile=Win10x64 procdump -p 1337 -D ./out/
```

## 4. Disk imaging and file system analysis

```bash
# Bit-for-bit forensic image, write-blocked source
sudo dd if=/dev/sdb of=evidence.img bs=4M status=progress conv=sync,noerror

# Or, purpose-built (adds metadata + verification)
sudo dc3dd if=/dev/sdb hash=sha256 log=acquire.log of=evidence.img
```

Once imaged, mount read-only for examination, or use `Autopsy` /
`The Sleuth Kit` to browse the file system without altering it:

```bash
sudo mount -o ro,loop evidence.img /mnt/evidence
fls -r -m / evidence.img > bodyfile.txt   # timeline of all file activity
```

## 5. Timeline analysis

Reconstructing "what happened when" is often the single most valuable
forensic artifact, correlating file system timestamps, log entries, and
memory artifacts into one sequence:

```bash
mactime -b bodyfile.txt -d > timeline.csv
```

A timeline turns scattered clues ("a file was modified," "a process
started," "a login occurred") into a story: *user account compromised at
14:02 → malicious scheduled task created at 14:03 → data staged to
C:\Temp at 14:07 → exfiltration connection at 14:09.*

## 6. Windows artifacts every analyst should know

| Artifact | What it shows |
|---|---|
| Prefetch (`C:\Windows\Prefetch`) | Programs that were executed, and when |
| `$MFT` (Master File Table) | File creation/modification even after deletion |
| Registry `NTUSER.DAT`, `SYSTEM`, `SAM` | User activity, USB history, installed software |
| Event Logs (Security, System) | Logons, service starts, policy changes |
| ShimCache / AmCache | Historical evidence of executed binaries |

```powershell
# Pull recent Security event log logon events (Event ID 4624/4625) for
# triage without a full SIEM
Get-WinEvent -FilterHashtable @{LogName='Security'; Id=4624,4625} -MaxEvents 50
```

## 7. Reporting

A forensic report must let another qualified analyst reproduce your
conclusions: methodology, tools and versions used, evidence hashes at
each stage, findings, and a clear separation between **fact** ("event
X occurred at timestamp Y per artifact Z") and **interpretation**
("this is consistent with data staging prior to exfiltration").

## How It Actually Works: how a disk image is bit-for-bit verified, and how a filesystem timeline is actually reconstructed

**Disk imaging integrity** rests on cryptographic hashing exactly as
described in Level 1 Module 4, applied at forensic scale: the examiner hashes
the source drive (commonly with SHA-256, historically MD5) *before*
acquisition, then hashes the resulting image file after, and the two must
match exactly. Because a hash function's avalanche property means a single
flipped bit anywhere in gigabytes of data changes roughly half the output
bits, a matching hash is strong evidence — not just documentation — that the
image is a perfect bit-for-bit copy, including deleted files, filesystem
metadata, and slack space, not merely the "logical" files a normal copy
would preserve. This is why imaging tools use a **write blocker** at the
hardware or driver level: it physically or logically intercepts every write
command sent to the source drive and drops it, so the acquisition process
itself cannot alter the evidence it's trying to preserve — without it, even
mounting the drive read-only in some OSes can trigger journal replay writes
that change the very hash you're trying to preserve unmodified.

**Timeline analysis** works because filesystems record more temporal
metadata than most people realize. NTFS, for instance, keeps eight
timestamps per file across two attribute records (`$STANDARD_INFORMATION`
and `$FILE_NAME`), each with its own Modified/Accessed/Created/Entry-modified
values — and critically, the two records update under *different* rules
(`$STANDARD_INFORMATION` timestamps can be trivially altered by
user-space "timestomping" tools using documented Windows APIs, but
`$FILE_NAME` timestamps are only updated by the NTFS driver itself during
specific low-level operations). An analyst builds a **super-timeline** by
extracting every timestamp from every available source — filesystem
metadata, the Windows Event Log (itself a chain of timestamped, sequentially
numbered records, making gaps detectable), registry key last-write times,
browser history — and sorting the union chronologically. Log entries and
artifacts corroborate each other because a genuine sequence of actions
leaves consistent traces across *independent* subsystems that an attacker
would need to separately falsify; a mismatch between what
`$STANDARD_INFORMATION` claims and what the (harder-to-forge)
`$FILE_NAME` record or Event Log shows for the same file is itself forensic
evidence of tampering, which is exactly how timestomping gets detected
rather than simply trusted.

## 8. Checklist

- [ ] Evidence hashed before analysis; hash re-verified before reporting
- [ ] Chain of custody log started and maintained
- [ ] Volatile evidence (RAM, network state) captured before power-off
- [ ] All analysis performed on a copy, never the original
- [ ] Timeline built from at least two independent artifact sources
- [ ] Report separates verified fact from analyst interpretation

## What's next

Module 3 covers analyzing the malware itself once forensics has located
the sample.
