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
