---
title: "Final Quiz"
description: "Test what you have learned across all four levels of Cybersecurity Mastery Path with a 20-question final quiz."
---

# Final Quiz

20 questions drawn from all four levels of this course. You need **70% (14 out of 20)** to pass —
you can retake it as many times as you like. Your best result is saved in this browser.

<div id="mp-quiz-root"></div>

<script type="application/json" id="mp-quiz-data">
[
  {
    "question": "A hospital's files are encrypted by malware and staff can't open them. Which CIA property is hit most directly?",
    "options": [
      "Integrity only",
      "Authentication",
      "Confidentiality",
      "Availability"
    ],
    "correctIndex": 3,
    "explanation": "The data exists but isn't usable."
  },
  {
    "question": "Which description best separates a threat from a vulnerability?",
    "options": [
      "A threat is a patch",
      "A vulnerability is an attacker",
      "A threat is something that could exploit a weakness; a vulnerability is the weakness itself",
      "They're identical"
    ],
    "correctIndex": 2,
    "explanation": "Risk combines likelihood and impact."
  },
  {
    "question": "How many usable hosts are in 10.0.0.0/22?\n```python\nimport ipaddress\nprint(ipaddress.ip_network('10.0.0.0/22').num_addresses-2)\n```\n",
    "options": [
      "1024",
      "1020",
      "254",
      "1022"
    ],
    "correctIndex": 3,
    "explanation": "2^10 addresses minus network and broadcast. Verified by running it."
  },
  {
    "question": "A user submits `x' OR '1'='1` into a query built by string concatenation versus a parameterized query. What do the two counts show?\n```python\nimport sqlite3\nc=sqlite3.connect(':memory:'); c.execute('create table u(n)'); c.execute(\"insert into u values('ada')\")\nv=\"x' OR '1'='1\"\nprint(c.execute(\"select count(*) from u where n='\"+v+\"'\").fetchone()[0], c.execute('select count(*) from u where n=?',(v,)).fetchone()[0])\n```\n",
    "options": [
      "1 0",
      "0 0",
      "1 1",
      "0 1"
    ],
    "correctIndex": 0,
    "explanation": "Parameterization keeps input as data. Use it everywhere. Verified by running it."
  },
  {
    "question": "An IDOR lets a user view /invoice/1002 by changing the number from 1001. What's the fix?",
    "options": [
      "Obfuscate IDs only",
      "Enforce authorization server-side on each request (check ownership), not just hide links",
      "Use POST",
      "Hide the URL"
    ],
    "correctIndex": 1,
    "explanation": "Broken access control is the top OWASP risk."
  },
  {
    "question": "Why hash passwords with salted, slow algorithms such as Argon2 or bcrypt?",
    "options": [
      "It saves space",
      "Hashes can be decrypted",
      "Salts encrypt",
      "Salts defeat precomputed tables and slow hashing makes mass guessing costly"
    ],
    "correctIndex": 3,
    "explanation": "Never use fast unsalted hashes."
  },
  {
    "question": "Phishing-resistant MFA is best implemented with...",
    "options": [
      "Security questions",
      "SMS codes",
      "Email codes",
      "FIDO2/WebAuthn hardware keys or passkeys"
    ],
    "correctIndex": 3,
    "explanation": "They bind to the real site."
  },
  {
    "question": "A user reports a suspicious email. What should the SOC do first?",
    "options": [
      "Click to test",
      "Delete silently",
      "Check headers and links safely, search for other recipients and block indicators",
      "Forward to all"
    ],
    "correctIndex": 2,
    "explanation": "Contain and hunt."
  },
  {
    "question": "Which log pattern suggests a successful brute-force attack?",
    "options": [
      "No logs",
      "Many failures from one IP followed by a success",
      "One success",
      "Normal logins"
    ],
    "correctIndex": 1,
    "explanation": "Correlation rule."
  },
  {
    "question": "During an incident, why contain before eradicating?",
    "options": [
      "To stop spread and preserve evidence while you analyze",
      "Style",
      "Speed",
      "Cost"
    ],
    "correctIndex": 0,
    "explanation": "Isolate affected systems."
  },
  {
    "question": "When making a forensic copy of a disk, why hash it?",
    "options": [
      "Compress",
      "To prove the evidence hasn't changed",
      "Encrypt",
      "Rename"
    ],
    "correctIndex": 1,
    "explanation": "Document chain of custody."
  },
  {
    "question": "A cloud storage bucket is publicly readable by mistake. Who is responsible for that configuration?",
    "options": [
      "Nobody",
      "The customer, under the shared responsibility model",
      "The ISP",
      "The provider"
    ],
    "correctIndex": 1,
    "explanation": "Misconfiguration is the leading cause of cloud breaches."
  },
  {
    "question": "A pentest is requested. What must exist before testing starts?",
    "options": [
      "A verbal OK",
      "A report template",
      "Written authorization with scope and Rules of Engagement",
      "A tool"
    ],
    "correctIndex": 2,
    "explanation": "Otherwise it's unauthorized access."
  },
  {
    "question": "A vulnerability scanner reports a critical CVE on a server not reachable from the internet. How should you prioritize?",
    "options": [
      "Patch by score only",
      "Combine CVSS with context: exposure, exploitability and asset value",
      "Ignore",
      "Delete the server"
    ],
    "correctIndex": 1,
    "explanation": "Risk-based prioritization."
  },
  {
    "question": "Which is a zero-trust design choice?",
    "options": [
      "One flat VPN",
      "Per-request verification of identity and device posture with least-privilege access",
      "Shared admin accounts",
      "Trust the internal network"
    ],
    "correctIndex": 1,
    "explanation": "Never trust, always verify."
  },
  {
    "question": "Containers run as root with host mounts. What's the risk and fix?",
    "options": [
      "No risk",
      "A compromise can escape to the host; run non-root, drop capabilities and avoid broad mounts",
      "Only logs",
      "Speed"
    ],
    "correctIndex": 1,
    "explanation": "Least privilege for workloads."
  },
  {
    "question": "A detection rule fires 500 false alerts a day. What do you do?",
    "options": [
      "Tune it with context and thresholds, and measure precision",
      "Ignore",
      "Add alerts",
      "Disable all"
    ],
    "correctIndex": 0,
    "explanation": "Alert fatigue hides real attacks."
  },
  {
    "question": "An executive asks 'are we secure?'. How should you answer?",
    "options": [
      "No",
      "Technical details",
      "Yes",
      "Express residual risk in business terms with evidence, not a yes or no"
    ],
    "correctIndex": 3,
    "explanation": "Risk management."
  },
  {
    "question": "Which compliance statement is accurate?",
    "options": [
      "Compliance sets a baseline but doesn't guarantee security",
      "Compliant means secure",
      "Audits stop attacks",
      "Controls aren't needed"
    ],
    "correctIndex": 0,
    "explanation": "Evidence of effective operation matters."
  },
  {
    "question": "Why plan a security roadmap in phases?",
    "options": [
      "Resources are limited, so fix highest risks first and show measurable progress",
      "Law",
      "Style",
      "Speed"
    ],
    "correctIndex": 0,
    "explanation": "Prioritize by risk."
  }
]
</script>

## Coding challenges

Try these on your own computer, then compare your output. Nothing you write is sent anywhere.

### Store and verify passwords safely

Implement hash_password() with PBKDF2-HMAC-SHA256 and verify() using a constant-time comparison. Show that the right password verifies, a similar one fails, and that a different salt yields a different hash.

**Starter code**

```python
import hashlib, hmac

def hash_password(password, salt, rounds=100_000):
    # derive a key with PBKDF2-HMAC-SHA256 and return it as a hex string
    return ""

def verify(password, salt, stored, rounds=100_000):
    # recompute and compare in constant time (hmac.compare_digest)
    return False

salt = bytes.fromhex("00112233445566778899aabbccddeeff")
stored = hash_password("correct horse", salt)
print(len(stored), stored[:16])
print(verify("correct horse", salt, stored))
print(verify("Correct horse", salt, stored))
print(hash_password("correct horse", salt) == hash_password("correct horse", bytes(16)))
```

**Expected output**

```text
64 349385d8369097aa
True
False
False
```

<details><summary>Show solution and explanation</summary>

<pre><code>import hashlib, hmac

def hash_password(password, salt, rounds=100_000):
    return hashlib.pbkdf2_hmac(&quot;sha256&quot;, password.encode(), salt, rounds).hex()

def verify(password, salt, stored, rounds=100_000):
    candidate = hash_password(password, salt, rounds)
    return hmac.compare_digest(candidate, stored)

salt = bytes.fromhex(&quot;00112233445566778899aabbccddeeff&quot;)
stored = hash_password(&quot;correct horse&quot;, salt)
print(len(stored), stored[:16])
print(verify(&quot;correct horse&quot;, salt, stored))
print(verify(&quot;Correct horse&quot;, salt, stored))
print(hash_password(&quot;correct horse&quot;, salt) == hash_password(&quot;correct horse&quot;, bytes(16)))
</code></pre>

<p>A unique random salt per user and a slow key-derivation function make offline guessing expensive, and compare_digest avoids timing leaks. The salt here is fixed only so the output is reproducible. Verified by running the solution.</p>
</details>

### Detect a brute-force that succeeded

Parse SSH auth log lines and flag a source IP only when a successful login follows at least `threshold` failures from that IP. Return {ip: failures_before_success}. A separate IP with just two failures must not be flagged.

**Starter code**

```python
import re
from collections import defaultdict

LOG = """\
Nov 21 10:00:01 sshd[1]: Failed password for root from 203.0.113.7 port 51111 ssh2
Nov 21 10:00:03 sshd[1]: Failed password for admin from 203.0.113.7 port 51112 ssh2
Nov 21 10:00:05 sshd[1]: Failed password for root from 203.0.113.7 port 51113 ssh2
Nov 21 10:00:07 sshd[1]: Failed password for ada from 198.51.100.4 port 40001 ssh2
Nov 21 10:00:08 sshd[1]: Failed password for root from 203.0.113.7 port 51114 ssh2
Nov 21 10:00:09 sshd[1]: Accepted password for root from 203.0.113.7 port 51115 ssh2
Nov 21 10:03:00 sshd[1]: Failed password for ada from 198.51.100.4 port 40002 ssh2
"""

PATTERN = re.compile(r"(?P<status>Failed|Accepted) password for (?P<user>\S+) from (?P<ip>\S+)")

def suspicious(log, threshold=4):
    # count failed logins per source IP; flag an IP only if a successful login
    # follows at least `threshold` failures. Return {ip: failures_before_success}.
    return {}

print(suspicious(LOG))
```

**Expected output**

```text
{'203.0.113.7': 4}
```

<details><summary>Show solution and explanation</summary>

<pre><code>import re
from collections import defaultdict

LOG = &quot;&quot;&quot;\
Nov 21 10:00:01 sshd[1]: Failed password for root from 203.0.113.7 port 51111 ssh2
Nov 21 10:00:03 sshd[1]: Failed password for admin from 203.0.113.7 port 51112 ssh2
Nov 21 10:00:05 sshd[1]: Failed password for root from 203.0.113.7 port 51113 ssh2
Nov 21 10:00:07 sshd[1]: Failed password for ada from 198.51.100.4 port 40001 ssh2
Nov 21 10:00:08 sshd[1]: Failed password for root from 203.0.113.7 port 51114 ssh2
Nov 21 10:00:09 sshd[1]: Accepted password for root from 203.0.113.7 port 51115 ssh2
Nov 21 10:03:00 sshd[1]: Failed password for ada from 198.51.100.4 port 40002 ssh2
&quot;&quot;&quot;

PATTERN = re.compile(r&quot;(?P&lt;status&gt;Failed|Accepted) password for (?P&lt;user&gt;\S+) from (?P&lt;ip&gt;\S+)&quot;)

def suspicious(log, threshold=4):
    fails = defaultdict(int)
    flagged = {}
    for line in log.splitlines():
        m = PATTERN.search(line)
        if not m:
            continue
        ip = m[&quot;ip&quot;]
        if m[&quot;status&quot;] == &quot;Failed&quot;:
            fails[ip] += 1
        elif fails[ip] &gt;= threshold:
            flagged[ip] = fails[ip]
    return flagged

print(suspicious(LOG))
</code></pre>

<p>Counting failures per source and alerting on a success after a burst is the classic brute-force correlation rule. In a SIEM you&#x27;d also window it by time and enrich it. Verified by running the solution.</p>
</details>
