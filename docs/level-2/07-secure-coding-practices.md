# 07 · Secure Coding Practices

Level 1 Module 6 and Level 2 Module 2 showed injection and XSS as *symptoms*.
This module covers the *discipline* that prevents them: secure coding
practices that apply regardless of language or framework, plus concrete
patterns to use (and anti-patterns to avoid) in real code.

## 1. The core principle: never trust input

Every piece of data that crosses a trust boundary — a form field, a URL
parameter, an HTTP header, a file upload, a message from another service —
must be treated as untrusted until validated.

```python
# WRONG: trusts that "amount" is always a well-formed positive number
amount = request.form["amount"]
balance -= float(amount)

# RIGHT: validate type, range, and business rules before using
try:
    amount = float(request.form["amount"])
except ValueError:
    abort(400, "amount must be numeric")
if amount <= 0 or amount > balance:
    abort(400, "invalid amount")
balance -= amount
```

## 2. Input validation: allowlist over denylist

```python
# WRONG (denylist): trying to block "bad" characters is a losing game --
# you will always miss an encoding or edge case
if "<script>" in user_input:
    reject()

# RIGHT (allowlist): define exactly what's valid, reject everything else
import re
if not re.fullmatch(r"[A-Za-z0-9_-]{1,32}", username):
    reject()
```

Allowlisting is strictly stronger because it doesn't require anticipating
every possible attack encoding — it only requires knowing what legitimate
input looks like.

## 3. Output encoding — context matters

The same untrusted string needs *different* encoding depending on where
it's placed:

```python
from markupsafe import escape
import json, urllib.parse

user_input = '<script>alert(1)</script>'

html_context = f"<p>{escape(user_input)}</p>"                # HTML-escape
js_context = f"var x = {json.dumps(user_input)};"             # JSON-encode
url_context = f"/search?q={urllib.parse.quote(user_input)}"   # URL-encode
```

Using the wrong encoding for the context (e.g. HTML-escaping something
placed inside a `<script>` block) is itself a common source of "fixed but
still vulnerable" bugs.

## 4. Parameterized queries and ORMs (recap + generalization)

```python
# Every ORM does this correctly by default -- the vulnerability in Level 1
# Module 6 only existed because the demo bypassed the ORM with raw SQL
User.query.filter_by(username=username).first()          # SQLAlchemy, safe
db.execute("SELECT * FROM users WHERE username = ?", (username,))  # safe
```

The same pattern generalizes to any interpreter: shell commands, LDAP
queries, XML/XPath, NoSQL query objects — anywhere untrusted data could
alter the *structure* of a command rather than just its *value*.

```python
# Command injection -- WRONG
os.system(f"ping -c 1 {host}")
# Command injection -- RIGHT: no shell, arguments passed as a list
subprocess.run(["ping", "-c", "1", host], shell=False)
```

## 5. Authentication and secrets in code

```python
# WRONG: hardcoded secret, committed to version control forever
API_KEY = "sk-live-abc123..."

# RIGHT: loaded from environment/secret manager, never in source
import os
API_KEY = os.environ["API_KEY"]
```

```bash
# Scan a repo for accidentally-committed secrets before every push
pip install detect-secrets
detect-secrets scan --all-files
```

Password storage: never store plaintext or reversibly-encrypted
passwords — use a slow, salted hash designed for the purpose:

```python
# WRONG
password_hash = hashlib.md5(password.encode()).hexdigest()   # fast, broken

# RIGHT
import bcrypt
password_hash = bcrypt.hashpw(password.encode(), bcrypt.gensalt())
bcrypt.checkpw(password.encode(), password_hash)   # for verification
```

`bcrypt`/`argon2`/`scrypt` are deliberately slow and salted, which makes
offline cracking of a stolen hash database computationally expensive —
fast general-purpose hashes (MD5, SHA-1, even unsalted SHA-256) are
unsuitable for passwords specifically because they're *too fast*.

## 6. Dependency and supply chain hygiene

```bash
# Check for known-vulnerable dependencies before every release
pip install pip-audit && pip-audit
npm audit
```

Pin dependency versions and review changelogs on updates — a supply-chain
compromise of a popular package (a real, recurring attack pattern) can
inject malicious code into thousands of downstream apps that "just ran
`npm install`."

## 7. Secure code review checklist

Use this checklist on every pull request touching security-relevant code:

- [ ] Is every external input validated (type, length, format, range)?
- [ ] Is all output encoded for the context it's rendered into?
- [ ] Are all database/shell/LDAP queries parameterized, never string-built?
- [ ] Are secrets loaded from environment/vault, never hardcoded?
- [ ] Are passwords hashed with bcrypt/argon2, never a fast general hash?
- [ ] Does error handling avoid leaking stack traces/internals to users?
- [ ] Are authorization checks present on every endpoint that needs them
      (not just authentication — being logged in ≠ being allowed to act)?
- [ ] Are file uploads validated by content, not just filename/extension?

## 8. Static analysis: automate what the checklist covers

```bash
# Python
pip install bandit && bandit -r .

# JavaScript
npm install --save-dev eslint-plugin-security
```

Static Application Security Testing (SAST) tools like Bandit and
Semgrep catch a large fraction of the checklist automatically, on every
commit, in CI — freeing manual review to focus on business logic and
authorization, which tools can't reason about.

## Key terms

| Term | Meaning |
|---|---|
| **Trust boundary** | Any point where data crosses from an untrusted source into your system |
| **Allowlist / denylist** | Defining what's explicitly permitted vs. trying to block what's forbidden |
| **Output encoding** | Transforming data for the specific context (HTML/JS/URL) it's rendered into |
| **SAST** | Static Application Security Testing — analyzing source code for vulnerabilities without running it |
| **Supply chain attack** | Compromise introduced via a trusted third-party dependency |
| **Slow hash** | A deliberately computation-expensive hash function (bcrypt/argon2) used for password storage |

## Exercise

1. Take the vulnerable Flask app from Level 1 Module 6 and run `bandit -r .`
   against it — confirm it flags the raw SQL string-building.
2. Fix the flagged issues using the patterns in sections 3–4, then re-run
   Bandit to confirm a clean report.
3. Add `bcrypt` password hashing to the app's login flow, replacing any
   plaintext comparison.
4. Run `detect-secrets scan` against a repo of your choice and review
   (don't necessarily fix) whatever it flags.
5. Written answer: explain, with a concrete example, why "being
   authenticated" and "being authorized" are different checks that both
   need to happen on every sensitive endpoint.
