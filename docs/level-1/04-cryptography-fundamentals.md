# 04 · Cryptography Fundamentals

Cryptography is how confidentiality and integrity (Module 1's CIA triad)
actually get *enforced* mathematically, rather than just hoped for. This
module covers the three building blocks you'll see everywhere else in this
course — symmetric encryption, asymmetric encryption, and hashing — and how
they combine to form TLS, the protocol behind every `https://` URL.

## 1. Symmetric encryption — one shared key

Symmetric encryption uses the **same key** to encrypt and decrypt. It's fast
and simple, with one hard problem: both parties need the same secret key
*before* they can communicate securely, which means securely delivering that
key in the first place is a separate, unsolved problem.

```
plaintext --[encrypt with key K]--> ciphertext --[decrypt with key K]--> plaintext
```

| Algorithm | Key size | Status |
|---|---|---|
| **AES** (Advanced Encryption Standard) | 128/192/256-bit | Current standard — use this |
| DES | 56-bit | Broken (brute-forceable), do not use |
| 3DES | 112-bit effective | Deprecated, being phased out everywhere |
| RC4 | Variable | Broken, do not use |

AES is what actually encrypts your disk (BitLocker, FileVault), your Wi-Fi
traffic (WPA2/WPA3), and the bulk of data inside a TLS connection (section 4).
It's fast enough to encrypt gigabytes per second on ordinary hardware, which
is exactly why it's used for the bulk data rather than asymmetric crypto.

## 2. Asymmetric encryption — two related keys

Asymmetric (public-key) encryption uses a **key pair**: a public key you can
give to anyone, and a private key you never share. What one key encrypts, only
the other can decrypt.

```
Anyone can encrypt with Alice's PUBLIC key
     |
     v
Only Alice's PRIVATE key can decrypt it
```

This solves symmetric encryption's key-distribution problem: Alice publishes
her public key openly; anyone can send her a secret message that only she can
read, without ever needing a pre-shared secret.

| Algorithm | Typical use |
|---|---|
| **RSA** | Key exchange, digital signatures, still very common |
| **ECC / ECDSA** (Elliptic Curve) | Same purposes as RSA, smaller keys for equivalent security — the modern default |
| **Diffie-Hellman** | Key *exchange* specifically — letting two parties agree on a shared secret over an insecure channel |

Asymmetric crypto is roughly 100–1000x slower than symmetric crypto for the
same amount of data, which is why real systems never encrypt bulk data
asymmetrically — they use asymmetric crypto to securely exchange a
*symmetric* key, then switch to fast symmetric encryption for the actual
data. That handoff is exactly what happens in the TLS handshake (section 4).

### Digital signatures — the reverse use

Flip the direction: **sign** with your private key, anyone can **verify** with
your public key. This proves the message came from you (authenticity) and
wasn't altered (integrity) — it does not provide confidentiality by itself.

```
Alice signs a message with her PRIVATE key --> signature
Anyone verifies the signature with Alice's PUBLIC key --> confirms it's really from Alice, unaltered
```

This is how software updates are verified as genuinely from the vendor, how
HTTPS certificates are trusted (section 4), and how commits can be
cryptographically signed to prove authorship.

## 3. Hashing — one-way fingerprints

A hash function takes input of any size and produces a **fixed-size output**
(a "digest") such that the same input always produces the same output, a tiny
change in input produces a wildly different output, and — critically — you
cannot reverse a hash to recover the original input.

```bash
$ echo -n "hello" | sha256sum
2cf24dba5fb0a30e26e83b2ac5b9e29e1b161e5c1fa7425e73043362938b9824  -

$ echo -n "hellp" | sha256sum          # one letter changed
d0b3f0d7f38472b8f66e6f26c6e3d1e5b8e5cd6c72e5cf1a1ea3d5e2a7bfe789  -
```

That single-character change produces a completely different digest — this
property is called the **avalanche effect**, and it's what makes hashing
useful for integrity checks: if a file's hash doesn't match the expected
value, the file was altered (or corrupted), even by one bit.

| Algorithm | Status |
|---|---|
| MD5 | Broken — collisions are trivial to generate, never use for security |
| SHA-1 | Broken — deprecated for security purposes |
| **SHA-256 / SHA-3** | Current standard for general-purpose hashing |
| **bcrypt / Argon2 / scrypt** | Purpose-built for password hashing (see below) |

!!! warning "Never hash passwords with a general-purpose hash function alone"
    SHA-256 is *fast* — which is exactly the wrong property for password
    storage. A fast hash lets an attacker who steals your password database
    try billions of guesses per second (a "brute-force" or "dictionary"
    attack). Password-specific functions like **bcrypt**, **Argon2**, and
    **scrypt** are deliberately slow and memory-hard, and incorporate a
    **salt** — random data unique per password — so that two identical
    passwords never produce the same stored hash, defeating precomputed
    "rainbow table" lookups. Module 5 covers this in the context of
    authentication design.

### Hashing vs. encryption — the distinction beginners mix up

| | Hashing | Encryption |
|---|---|---|
| Reversible? | No — one-way by design | Yes — decrypt with the right key |
| Purpose | Verify integrity, store passwords | Protect confidentiality |
| Output size | Fixed, regardless of input size | Roughly proportional to input size |
| Example use | "Does this download match the vendor's checksum?" | "Only the recipient should be able to read this" |

## 4. Putting it together: TLS

**TLS (Transport Layer Security)** — what makes `https://` different from
`http://` — combines all three primitives above in one handshake:

```
1. Client Hello    -->  "Here are the TLS versions/ciphers I support"
2. Server Hello    <--  "Let's use TLS 1.3 with AES-256-GCM; here's my certificate"
                         (certificate contains the server's PUBLIC key,
                          signed by a trusted Certificate Authority)
3. Key exchange    <->  Client and server use asymmetric crypto (or
                         Diffie-Hellman) to agree on a shared SYMMETRIC key,
                         without ever transmitting that key in the clear
4. Verify          -->  Client checks the certificate: is it signed by a
                         trusted CA, is it for the right domain, is it
                         unexpired?
5. Secure channel  <->  All further traffic is encrypted with the fast
                         symmetric key from step 3, and integrity-checked
                         with a hash-based MAC on every message
```

This is precisely why the `curl -v` output from Module 2's exercise showed a
"TLS version negotiated" and a certificate issuer — you were watching this
handshake happen. A **Certificate Authority (CA)** is a trusted third party
that vouches "this public key really belongs to `example.com`" by digitally
signing the certificate; your browser trusts a pre-installed list of CAs, and
a certificate warning means either the certificate is self-signed, expired,
for the wrong domain, or signed by a CA your browser doesn't recognize — any
of which is a legitimate reason to stop and investigate rather than click
through.

!!! tip "Check a real certificate"
    Click the padlock icon in your browser's address bar on any HTTPS site and
    view the certificate. You'll see the domain it's issued for, the issuing
    CA, the validity dates, and (if you dig into details) the public key
    algorithm — RSA or ECDSA — all concepts from this module made concrete.

## Key terms

| Term | Meaning |
|---|---|
| **Symmetric encryption** | One shared key encrypts and decrypts (e.g., AES) |
| **Asymmetric encryption** | A public/private key pair; one encrypts, the other decrypts |
| **Hashing** | One-way fixed-size fingerprint of data (e.g., SHA-256) |
| **Salt** | Random per-password data mixed in before hashing, defeating precomputed attacks |
| **Digital signature** | Proof of authorship/integrity using a private key, verified with the public key |
| **Certificate Authority (CA)** | A trusted party that signs certificates binding a public key to an identity |
| **TLS** | The protocol combining all of the above to secure a connection (what `https://` uses) |

## Exercise

Use your terminal — `openssl` ships with macOS and most Linux distributions;
on Windows use WSL or Git Bash.

1. **Hash and verify.** Create a text file, compute its SHA-256 hash
   (`shasum -a 256 file.txt` on macOS, `sha256sum file.txt` on Linux), then
   change one character in the file and hash it again. Confirm the digests are
   completely different, and write one sentence on why this makes SHA-256
   useful for verifying a downloaded file wasn't tampered with in transit.

2. **Symmetric encryption round-trip.** Encrypt and decrypt a file with AES:
   ```bash
   openssl enc -aes-256-cbc -pbkdf2 -salt -in secret.txt -out secret.enc
   openssl enc -aes-256-cbc -pbkdf2 -d -salt -in secret.enc -out secret_decrypted.txt
   diff secret.txt secret_decrypted.txt   # should show no difference
   ```
   Then try decrypting with a deliberately wrong password and record what
   happens.

3. **Generate a key pair and sign something.**
   ```bash
   openssl genpkey -algorithm RSA -out private.pem -pkeyopt rsa_keygen_bits:2048
   openssl rsa -pubout -in private.pem -out public.pem
   openssl dgst -sha256 -sign private.pem -out signature.bin secret.txt
   openssl dgst -sha256 -verify public.pem -signature signature.bin secret.txt
   ```
   Confirm it prints `Verified OK`. Then modify `secret.txt` by one character
   and re-run the verify step — record what happens and explain why in terms
   of the avalanche effect from section 3.

4. **Inspect a real certificate.** Run
   `openssl s_client -connect example.com:443 -servername example.com </dev/null 2>/dev/null | openssl x509 -noout -issuer -dates -subject`
   and identify the issuing CA and the certificate's validity window.
