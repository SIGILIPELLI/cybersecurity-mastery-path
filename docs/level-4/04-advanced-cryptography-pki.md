# 04 · Advanced Cryptography & PKI

Level 1 Module 4 covered cryptography fundamentals — symmetric/asymmetric
encryption, hashing. This module builds the PKI infrastructure that makes
those primitives usable at organizational scale, and looks at where
cryptography is headed.

## 1. PKI components and the trust chain

```
Root CA (offline, air-gapped, signs only intermediate CAs)
  -> Intermediate CA (online, signs end-entity certificates)
    -> End-entity certificate (a server, a user, a device)
```

The root CA is kept offline specifically because its compromise would
undermine trust in every certificate ever issued beneath it — issuing
day-to-day certificates from an intermediate limits blast radius if that
intermediate is ever compromised or needs revocation.

```bash
# Generate a certificate signing request (CSR) for a server
openssl req -new -newkey rsa:4096 -nodes \
  -keyout server.key -out server.csr \
  -subj "/CN=app.example.internal"

# Verify a certificate chain
openssl verify -CAfile ca-chain.pem server.crt
```

## 2. Certificate validation and revocation

A certificate can become untrustworthy before its expiry (private key
compromise, decommissioned server). Two revocation mechanisms exist:

```
CRL (Certificate Revocation List) -- CA publishes a signed list of
  revoked serial numbers; clients must download and check it
OCSP (Online Certificate Status Protocol) -- client asks the CA in
  real time "is this specific certificate still valid?"
OCSP Stapling -- the server itself fetches and attaches a signed OCSP
  response to the TLS handshake, avoiding a separate client->CA round trip
  and the privacy/reliability issues of live OCSP lookups
```

## 3. Mutual TLS (mTLS)

Standard TLS authenticates the server to the client. mTLS authenticates
*both directions* — critical for zero trust (Module 3) service-to-service
communication where every call must prove its own identity:

```bash
# Server requires and validates a client certificate too
openssl s_server -cert server.crt -key server.key \
  -CAfile client-ca.pem -Verify 1
```

## 4. Key management at scale

The hardest part of cryptography in practice is rarely the algorithm —
it's protecting the keys:

```
HSM (Hardware Security Module) -- keys generated and used inside
  tamper-resistant hardware, never exported in plaintext
KMS (cloud Key Management Service) -- managed key lifecycle: generation,
  rotation, access control, audit logging, without operating your own HSM
Envelope encryption -- encrypt data with a fast local data key, then
  encrypt that data key with a KMS-managed master key -- avoids sending
  every byte of data through the KMS while keeping the master key
  centrally controlled and auditable
```

```bash
aws kms encrypt --key-id alias/app-master-key \
  --plaintext fileb://data_key.bin \
  --output text --query CiphertextBlob
```

## 5. Perfect forward secrecy

```
Without PFS: if the server's long-term private key is ever compromised,
  every past session recorded by an eavesdropper can be decrypted retroactively
With PFS (ephemeral Diffie-Hellman, ECDHE): each session uses a unique,
  temporary key that is discarded after use -- compromising the long-term
  key does not expose past traffic
```

Modern TLS 1.3 makes forward-secret cipher suites mandatory rather than
optional — a direct architectural response to years of retroactive
decryption incidents in older TLS deployments.

## 6. Post-quantum cryptography readiness

Sufficiently powerful quantum computers would break RSA and elliptic
curve cryptography (via Shor's algorithm) — not an immediate operational
threat today, but "harvest now, decrypt later" data theft means long-
lived sensitive data encrypted today is already at risk for the future.

```
NIST-standardized post-quantum algorithms (2024):
  ML-KEM (CRYSTALS-Kyber)  -- key encapsulation
  ML-DSA (CRYSTALS-Dilithium) -- digital signatures
```

Organizations with long-lived sensitive data (health records, government,
long-term secrets) are beginning hybrid deployments — classical + post-
quantum algorithms together — so that breaking one algorithm alone isn't
sufficient to compromise the data.

## 7. Common cryptographic implementation mistakes

```
- Rolling your own crypto instead of using vetted libraries/protocols
- Using ECB mode block cipher encryption (patterns leak through -- the
  classic "ECB penguin" visual example)
- Hardcoded or predictable initialization vectors (IVs) / nonces
- Comparing secrets with a non-constant-time comparison (timing attacks)
- Weak randomness -- using a non-cryptographic PRNG for key generation
```

```python
# Bad: timing side-channel possible
if user_provided_token == stored_token:
    ...

# Good: constant-time comparison
import hmac
if hmac.compare_digest(user_provided_token, stored_token):
    ...
```

## How It Actually Works: how certificate chain validation and revocation checking actually run, and why post-quantum crypto changes the math, not just the key size

Validating a certificate is a **recursive chain-of-signatures check**: a
leaf certificate is verified by using the issuing CA's public key to check
the signature over the leaf's contents — if that signature verifies, the
client next needs to trust the issuing CA's own certificate, which is
itself verified by *its* issuer's public key, and this repeats until the
chain reaches a **root CA certificate** already present in the client's
local trust store (installed by the OS/browser vendor, not fetched over the
network — the one link in the whole chain that has to be trusted
axiomatically rather than proven). Every step is exactly the digital
signature verification from Level 1 Module 4: decrypt the signature with the
issuer's public key, compare the result against an independently computed
hash of the certificate's contents. This recursive structure is precisely
why compromising *any single CA anywhere in the trusted root store* is
catastrophic industry-wide — that CA can then forge a validly-chaining
certificate for any domain, and every client's validation will succeed
because the mathematical chain checks out even though the certificate is
fraudulent.

**Revocation checking** exists because chain validation alone can't detect
"this certificate was valid but the private key has since been stolen." The
original mechanism, **CRLs** (Certificate Revocation Lists), had the CA
publish a periodically-updated signed list of revoked serial numbers — a
client checking a cert against a CRL is fundamentally a batch, stale-by-design
check (typically updated daily). **OCSP** replaced this with a live query:
"is serial number X still valid?" answered in real time by an OCSP
responder — but this leaks every site a user visits to that responder and
adds a network round-trip (and latency) to every TLS handshake, and fails
insecurely if the responder is unreachable ("soft-fail" was the historical
default). **OCSP stapling** fixes both: the *web server itself* periodically
fetches a signed, time-stamped OCSP response for its own certificate and
attaches ("staples") it directly to the TLS handshake, so the client
verifies a fresh, CA-signed non-revocation proof without ever contacting the
OCSP responder itself — privacy preserved, latency removed, because the
proof rides along with data the server was sending anyway.

**Post-quantum readiness** is not "bigger keys" — it's a wholesale algorithm
swap because Shor's algorithm, run on a sufficiently large quantum computer,
solves the specific mathematical problems (integer factorization for RSA,
discrete logarithm for Diffie-Hellman/ECDHE) that today's asymmetric
cryptography's security *entirely* depends on, in polynomial rather than
exponential time — no key size increase defends against an algorithm that
changes the complexity class of the underlying problem itself. NIST's
selected post-quantum algorithms (like CRYSTALS-Kyber for key exchange)
instead rest on **lattice-based problems** (finding short vectors in a
high-dimensional lattice), for which no efficient quantum algorithm is
currently known — which is exactly why "hybrid" deployments run a classical
ECDHE exchange *and* a lattice-based exchange in parallel and combine both
outputs into the session key: the session remains secure as long as at
least one of the two underlying hard problems holds, hedging against a
future break in either one alone.

## 8. Checklist

- [ ] Root CA kept offline; intermediates handle day-to-day issuance
- [ ] Certificate revocation via OCSP stapling, not client-side CRL fetches
- [ ] mTLS used for service-to-service auth in zero trust architectures
- [ ] Keys managed via HSM/KMS, never hardcoded or stored in plaintext
- [ ] TLS configuration enforces forward-secret cipher suites (TLS 1.3)
- [ ] Long-lived sensitive data assessed for post-quantum migration risk
- [ ] Cryptographic code uses vetted libraries, constant-time comparisons

## What's next

Module 5 automates much of the detection and response work built through
Level 3 into a SOAR pipeline.
