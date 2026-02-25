# Verifier Core

Deterministic verification kernel for Receipt Verification Spec v1.

**Language:** Rust

**Properties:**
- Stateless pure function: `verify(bundle, inputs) → VerificationReport`
- Deterministic: identical inputs produce byte-identical reports
- Offline-capable: no network, no secrets, no state
- Minimal trusted computing base

**Verification pipeline (fixed order):**
1. Parse + schema validation
2. Canonicalize (JCS / RFC 8785)
3. Hash derivations (SHA-256)
4. Signature verify (Ed25519, domain-separated)
5. Policy evaluation
6. Revocation evaluation (staleness semantics)
7. Registry snapshot evaluation (certified-only)
8. Emit VerificationReport

## Build

```bash
cargo build --release
cargo test
```

## Status

🚧 In development
