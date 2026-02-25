# Receipt Verification Spec v1

## Overview

This specification defines the wire format, cryptographic properties, verification semantics, and reason code taxonomy for deterministic receipt-based verification.

**Immutable.** Once published, V1 semantics do not change. Semantic changes require V2.

**Deterministic.** Same inputs produce byte-identical verification reports.

**Offline-capable.** Verification requires no network when trust store and revocation set are supplied locally.

**Content-addressed.** `bundle_id` and `attestation_id` are SHA-256 digests over canonical bytes.

---

## Schemas

| Schema | Description |
|---|---|
| `schemas/receipt_bundle.schema.json` | Receipt bundle wire format |
| `schemas/signed_envelope.schema.json` | Signed envelope (attestation + signature) |
| `schemas/attestation.schema.json` | Attestation payload |
| `schemas/policy_body.schema.json` | Policy definition |
| `schemas/verification_report.schema.json` | Verification output |
| `schemas/revocation_set.schema.json` | Signed revocation set |
| `schemas/registry_snapshot.schema.json` | Signed registry snapshot |
| `schemas/context_binding_gate.schema.json` | CI gate context binding |
| `schemas/context_binding_lti.schema.json` | LMS LTI context binding |
| `schemas/reason_codes.v1.json` | Authoritative reason code list |

## Conformance Vectors

| Directory | Coverage |
|---|---|
| `vectors/canon/` | Canonicalization — acceptance and rejection cases |
| `vectors/signature/` | Signature verification — valid, invalid, malleability |
| `vectors/revocation/` | Revocation staleness — fail-open and fail-closed |
| `vectors/registry/` | Certified-only enforcement |
| `vectors/policy/` | Policy evaluation |
| `vectors/e2e/` | Full pipeline end-to-end |

Vector format:
```json
{
  "vector_id": "canon_reject_duplicate_key_001",
  "description": "Reject JSON with duplicate object keys",
  "input": {},
  "expected_verdict": "INVALID",
  "expected_reason_codes": ["ERR_CANONICALIZATION_FAILED"],
  "expected_report_digest": "..."
}
```

## Cryptographic Specification

### Canonicalization
- JSON Canonicalization Scheme (RFC 8785 / JCS)
- UTF-8, keys sorted lexicographically by Unicode code points, no whitespace
- **Rejects:** duplicate keys, floating point numbers, non-UTF-8

### Signatures
- Ed25519 only (V1)
- Domain separation: `VALORIN_ATTESTATION_V1 || 0x00 || canonical_bytes(payload)`
- Covers payload canonical bytes only

### Hashing
- SHA-256 for all content addressing
- `attestation_id = SHA-256(canonical(payload))`
- `bundle_id = SHA-256(canonical(bundle_without_bundle_id))`
- `policy_hash = SHA-256(canonical(policy_body))`

### Revocation
- Epoch-monotonic signed sets with `valid_until` staleness
- Policy-selected behavior: `FAIL_CLOSED` or `FAIL_OPEN`
- Targets: issuer keys, cert artifacts, tool versions

## Verification Pipeline

Fixed order. Not configurable.

```
1. Parse + schema validation
2. Canonicalize (JCS)
3. Hash derivations (SHA-256)
4. Signature verify (Ed25519, domain-separated)
5. Policy evaluation
6. Revocation evaluation (staleness semantics)
7. Registry snapshot evaluation (certified-only gating)
8. Emit VerificationReport (deterministically ordered reason codes)
```

## Reason Codes

Stable semantics. Deterministic ordering: stage → code (lexicographic) → object_ref (lexicographic).

Downstream systems can branch on specific codes, not just pass/fail.

See `schemas/reason_codes.v1.json` for the authoritative list.

## Enums

### Verdict
`VALID` · `INVALID` · `VALID_WITH_WARNINGS`

### AttestationType
`LMS_SESSION_V1` · `CI_GATE_V1` · `TOOL_CONFORMANCE_V1` · `REGISTRY_SNAPSHOT_V1` · `AGENT_RUN_V1` · `EVIDENCE_POINTER_V1`

### CertificationTier
`CONFORMANCE` · `GOVERNANCE` · `PRIVACY`

### CertificationStatus
`REQUESTED` → `UNDER_REVIEW` → `CERTIFIED` → `SUSPENDED` | `REVOKED` | `EXPIRED`

### EvidenceTier
`MINIMAL` (default, no raw content) · `STANDARD` (richer aggregates) · `STRICT` (raw content in evidence plane only)

### EnforcementMode
`RECEIPT_REQUIRED` · `RECEIPT_OPTIONAL`

### RevocationFailBehavior
`FAIL_CLOSED` · `FAIL_OPEN`
