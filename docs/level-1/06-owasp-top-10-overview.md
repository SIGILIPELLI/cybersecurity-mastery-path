# 06 · Common Web Vulnerabilities — OWASP Top 10 Overview

The **OWASP Top 10** is the industry-standard list of the most critical web
application security risks, maintained by the Open Web Application Security
Project and updated periodically from real-world breach data. This module
covers the concepts, then has you build and run a small, deliberately
vulnerable local Flask app so you can see two of the most common
vulnerabilities — SQL injection and XSS — actually happen, and actually get
fixed, on your own machine.

!!! warning "Scope: your own machine, offline, nothing else"
    Everything in this module runs on `127.0.0.1` (localhost) only. You will
    never point these techniques at a website, server, or app you don't own.
    Testing for these vulnerabilities against systems you don't have explicit
    written authorization to test is illegal in most jurisdictions — this
    module exists so you understand the mechanism defensively, the same way
    Security+ and CEH teach it.

## 1. The OWASP Top 10 (2021 edition), briefly

| # | Category | In short |
|---|---|---|
| A01 | **Broken Access Control** | An authenticated user can act beyond their intended permissions (Module 5) |
| A02 | **Cryptographic Failures** | Sensitive data exposed due to weak/missing encryption (Module 4) |
| A03 | **Injection** | Untrusted input is interpreted as code/commands (SQL injection, this module) |
| A04 | **Insecure Design** | The flaw is architectural, not a coding bug — no amount of patching fixes a design that never considered abuse |
| A05 | **Security Misconfiguration** | Default credentials, verbose error messages, unnecessary features left enabled |
| A06 | **Vulnerable and Outdated Components** | Using a library/framework with a known, unpatched vulnerability |
| A07 | **Identification and Authentication Failures** | Weak login/session mechanisms (Module 5) |
| A08 | **Software and Data Integrity Failures** | Trusting unsigned code/updates/CI pipelines |
| A09 | **Security Logging and Monitoring Failures** | Attacks go undetected because nobody's watching (Module 9) |
| A10 | **Server-Side Request Forgery (SSRF)** | Tricking a server into making requests on the attacker's behalf |

This module builds a hands-on demo of **A03 (Injection)**, specifically SQL
injection, and cross-site scripting (XSS) — technically its own category in
earlier OWASP editions and still one of the most common bugs found in the
wild, now folded conceptually under injection-style input-handling failures.

## 2. The vulnerable app

Create a file called `vulnerable_app.py`:

```python
"""
DELIBERATELY VULNERABLE demo app -- for local, offline, educational use only.
Do not deploy this. Do not expose it to a network. Localhost only.
"""
from flask import Flask, request
import sqlite3

app = Flask(__name__)
DB = "/tmp/vulnapp_demo.db"

def init_db():
    conn = sqlite3.connect(DB)
    conn.execute("DROP TABLE IF EXISTS users")
    conn.execute("CREATE TABLE users (id INTEGER PRIMARY KEY, username TEXT, password TEXT)")
    conn.execute("INSERT INTO users (username, password) VALUES ('admin', 's3cr3t-admin-pw')")
    conn.execute("INSERT INTO users (username, password) VALUES ('alice', 'alicepw123')")
    conn.commit()
    conn.close()

@app.route("/vulnerable_login", methods=["GET"])
def vulnerable_login():
    username = request.args.get("username", "")
    password = request.args.get("password", "")
    conn = sqlite3.connect(DB)
    # VULNERABLE: raw string interpolation into SQL -- classic SQL injection
    query = f"SELECT * FROM users WHERE username = '{username}' AND password = '{password}'"
    cur = conn.execute(query)
    row = cur.fetchone()
    conn.close()
    if row:
        return f"Login OK, welcome {row[1]} (query was: {query})"
    return f"Login failed (query was: {query})"

@app.route("/safe_login", methods=["GET"])
def safe_login():
    username = request.args.get("username", "")
    password = request.args.get("password", "")
    conn = sqlite3.connect(DB)
    # SAFE: parameterized query -- user input never becomes part of the SQL text
    cur = conn.execute("SELECT * FROM users WHERE username = ? AND password = ?", (username, password))
    row = cur.fetchone()
    conn.close()
    if row:
        return f"Login OK, welcome {row[1]}"
    return "Login failed"

@app.route("/vulnerable_greet")
def vulnerable_greet():
    name = request.args.get("name", "guest")
    # VULNERABLE: user input echoed directly into HTML -- reflected XSS
    return f"<html><body><h1>Hello, {name}!</h1></body></html>"

@app.route("/safe_greet")
def safe_greet():
    from markupsafe import escape
    name = request.args.get("name", "guest")
    # SAFE: escape user input before embedding in HTML
    return f"<html><body><h1>Hello, {escape(name)}!</h1></body></html>"

if __name__ == "__main__":
    init_db()
    app.run(host="127.0.0.1", port=5055)
```

Install Flask and run it:

```bash
pip install flask
python vulnerable_app.py
```

```
 * Serving Flask app 'vulnerable_app'
 * Debug mode: off
 * Running on http://127.0.0.1:5055
Press CTRL+C to quit
```

## 3. SQL injection, demonstrated

The vulnerable endpoint builds a SQL query by directly inserting the request
parameters into a string. A normal login attempt looks fine:

```bash
curl -s "http://127.0.0.1:5055/vulnerable_login?username=nope&password=nope"
```

```
Login failed (query was: SELECT * FROM users WHERE username = 'nope' AND password = 'nope')
```

Now supply a username designed to alter the query's *structure* rather than
just its data — a single quote to close the string early, `--` to comment out
the rest of the query (including the password check):

```bash
curl -s "http://127.0.0.1:5055/vulnerable_login?username=admin'--&password=x"
```

```
Login OK, welcome admin (query was: SELECT * FROM users WHERE username = 'admin'--' AND password = 'x')
```

**No password was needed.** The `--` turned everything after it into a SQL
comment, so the query the database actually executed was effectively
`SELECT * FROM users WHERE username = 'admin'` — the injected input changed
what *code* ran, not just what data was searched for. This is the entire
concept of injection in one example: input that was supposed to be *data*
got interpreted as *syntax*.

Now hit the safe version with the identical payload:

```bash
curl -s "http://127.0.0.1:5055/safe_login?username=admin'--&password=x"
```

```
Login failed
```

The **parameterized query** (`?` placeholders with values passed separately)
tells the database driver, unambiguously, "this entire string is a literal
value, never interpret it as SQL syntax" — regardless of what characters it
contains. This is the actual fix, not "block the `--` characters" or any
other input-filtering approach, which are fragile and routinely bypassed.
Parameterized queries (or an ORM that generates them for you) are the correct
defense against SQL injection, full stop.

## 4. Cross-site scripting (XSS), demonstrated

The vulnerable greet endpoint echoes a `name` parameter directly into an HTML
response with no escaping:

```bash
curl -s 'http://127.0.0.1:5055/vulnerable_greet?name=<script>alert(1)</script>'
```

```
<html><body><h1>Hello, <script>alert(1)</script>!</h1></body></html>
```

A real browser loading this URL would **execute** that `<script>` tag — this
is reflected XSS: an attacker crafts a URL containing malicious script,
tricks a victim into clicking it (phishing, Module 7), and the victim's own
browser runs the attacker's JavaScript in the context of the trusted site,
with access to that site's cookies and session.

The safe version escapes user input before embedding it:

```bash
curl -s 'http://127.0.0.1:5055/safe_greet?name=<script>alert(1)</script>'
```

```
<html><body><h1>Hello, &lt;script&gt;alert(1)&lt;/script&gt;!</h1></body></html>
```

`<` became `&lt;` and `>` became `&gt;` — the browser now renders the literal
text `<script>alert(1)</script>` as a harmless string instead of executing it
as markup. This is what every templating engine's "auto-escaping" mode
(Jinja2, which Flask uses by default when rendering templates properly,
React's JSX, etc.) does automatically — the vulnerability above only exists
because this demo builds HTML with raw f-strings instead of a template.

## 5. Why the fix generalizes

Both vulnerabilities above share one root cause: **untrusted input was mixed
with the code/markup that interprets it, without a boundary between the
two.** The fix in both cases is the same pattern — keep data and syntax
strictly separated:

| Vulnerability | Wrong (mixes data and syntax) | Right (keeps them separate) |
|---|---|---|
| SQL injection | String-building a query with f-strings/concatenation | Parameterized queries / prepared statements |
| XSS | Building HTML with raw string interpolation | Auto-escaping template engines, output encoding |
| Command injection (not demoed here) | `os.system(f"ping {user_input}")` | `subprocess.run(["ping", user_input])` — no shell interpretation |

This is the generalizable lesson Level 2 Module 2 and Level 2 Module 7 build
on: nearly every injection-class vulnerability is a variant of "we let
attacker-controlled text influence how a downstream interpreter parses
something."

## Key terms

| Term | Meaning |
|---|---|
| **Injection** | Untrusted input interpreted as code/commands rather than pure data |
| **SQL injection (SQLi)** | Injection targeting a SQL query |
| **XSS (Cross-Site Scripting)** | Injection of script that a victim's browser executes |
| **Parameterized query** | A query where data is passed separately from SQL syntax, preventing SQLi |
| **Output encoding / escaping** | Converting special characters so they render as text, not markup |
| **Reflected XSS** | Malicious script delivered via a crafted URL/request, executed once |

## Exercise

Using the app you just built and ran:

1. **Reproduce both attacks** exactly as shown above and capture the actual
   output on your machine — confirm the SQLi bypass logs in as `admin`
   without the password, and confirm the XSS payload appears unescaped in
   `/vulnerable_greet`.

2. **Try a second SQLi payload.** Instead of `admin'--`, try
   `' OR '1'='1`. Predict what the resulting query will look like before
   running it, then run it against `/vulnerable_login?username=' OR '1'='1&password=x`
   and check your prediction against the `query was:` text in the response.

3. **Confirm the fix holds** by running both of your payloads from steps 1–2
   against `/safe_login` and confirming both fail.

4. **Extend the app.** Add a third endpoint, `/vulnerable_search`, that takes
   a `q` parameter and returns `f"<p>Results for: {q}</p>"` — deliberately
   vulnerable to XSS like `/vulnerable_greet`. Then write `/safe_search`,
   its escaped counterpart, and prove both behave as expected with the same
   `<script>` payload used in section 4.

5. **Written answer.** In two or three sentences, explain why "just block
   quotes and semicolons" is not a real fix for SQL injection — think about
   what other characters or encodings might achieve the same syntactic break
   (hint: consider `OR`, `UNION`, or numeric-only injection with no quotes at
   all, e.g. against a query built as `... WHERE id = {user_input}`).
