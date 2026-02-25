# Architecture

## System Overview

```
┌─────────────────────────────────────────────────────┐
│                    OPEN SOURCE                       │
│                                                     │
│  ┌──────────────┐  ┌──────────┐  ┌──────────────┐  │
│  │ verifier-core│  │   CLI    │  │    SDKs      │  │
│  │  (Rust/WASM) │  │          │  │ (Rust / TS)  │  │
│  └──────┬───────┘  └────┬─────┘  └──────┬───────┘  │
│         │               │               │          │
│  ┌──────┴───────────────┴───────────────┴───────┐  │
│  │              Enforcement Gates                │  │
│  │  GitHub Action · GitLab CI · Canvas · D2L     │  │
│  └──────────────────────┬───────────────────────┘  │
│                         │                          │
│  ┌──────────────────────┴───────────────────────┐  │
│  │           Receipt Adapters                    │  │
│  │  Copilot · Cursor · Jest · pytest · ...       │  │
│  └──────────────────────────────────────────────┘  │
└─────────────────────────┬───────────────────────────┘
                          │ default authority
                          ▼
┌─────────────────────────────────────────────────────┐
│               AUTHORITY LAYER (hosted)               │
│                                                     │
│  ┌────────────┐ ┌────────────┐ ┌────────────────┐  │
│  │  Registry   │ │ Revocation │ │ Transparency   │  │
│  │ (cert auth) │ │ (kill sw.) │ │    Log         │  │
│  └────────────┘ └────────────┘ └────────────────┘  │
│                                                     │
│  ┌────────────┐ ┌────────────┐ ┌────────────────┐  │
│  │ Governance  │ │  Issuance  │ │   Autopilot    │  │
│  │ (approvals) │ │ (signing)  │ │ (containment)  │  │
│  └────────────┘ └────────────┘ └────────────────┘  │
└─────────────────────────────────────────────────────┘
```

## Trust Boundaries

| Boundary | Trust Level | What It Protects |
|---|---|---|
| Public Verify | Untrusted | Verification results. No auth. No evidence access. |
| Authenticated Tenant | Medium | Tenant-scoped receipts, policies, audit logs. JWT required. |
| Evidence Plane | High | Sealed evidence artifacts. Admin/auditor only. Reason-required access. |
| Signing Authority | Highest | KMS/HSM key handles. Issuance service only. |
| Revocation/Cert Authority | Highest | Registry and revocation decisions. Multi-party approval required. |

## Key Design Decisions

### Open Source Verifier + Proprietary Authority

The verifier is a pure function. Anyone can run it. It works offline with local trust stores and revocation sets. Open sourcing it eliminates "build vs. buy" analysis for platforms — there's nothing to build.

The authority layer (registry + revocation) is where the value concentrates. The registry determines which tools are certified. The revocation authority can deterministically invalidate receipts across every platform. This is a coordination asset, not a code asset — its value comes from who points to it, not how it's built.

### No Raw Content by Default

Receipts store policy hashes, commitments (bucketed counts/durations/lengths), and context bindings. Raw prompts, outputs, essays, and code are excluded from default receipt schemas.

This is a structural decision, not a policy. Schema validation rejects raw content fields. Log scanners block raw content in logs. DB roles prevent verify services from accessing the evidence plane.

The result: VALORIN can be deployed in privacy-sensitive environments (universities, regulated enterprises) without triggering data sensitivity review.

### Determinism as an Invariant

Verification is deterministic: identical inputs produce byte-identical reports. This is enforced by:

- JCS canonicalization (RFC 8785) — eliminates JSON ambiguity
- Rejecting floats and duplicate keys — eliminates parser disagreements
- Fixed verification pipeline order — eliminates evaluation order ambiguity
- Deterministic reason code ordering — eliminates output ambiguity
- Dual implementation conformance (Rust + WASM) — proves cross-platform consistency

Any nondeterminism event is a SEV0 incident that pauses issuance.

### Neutral Governance

The authority layer operates with strict neutrality:

- Non-exclusive: any platform can embed the verifier and use the authority
- Transparent: all certification and revocation actions are logged in an append-only transparency log
- Governed: revocation and certification changes require multi-party approval
- Narrow: "certified" means spec conformance, not security guarantee

This neutrality is what enables rival platforms to embed the same verification layer without ceding control to a competitor.

## Data Flow

### Receipt Issuance
```
Client → Issuance Service → KMS (sign) → DB (store) → Audit Log
```
No raw content. Receipt contains policy hash, commitments, context binding.

### Public Verification
```
Client → Verify API → Verifier Core → VerificationReport
```
Stateless. No auth. No evidence access. No receipt storage.

### Enforcement Gate (CI)
```
PR opened → Gate checks for receipt → Verify → Block or Allow merge
```
Anti-replay: server-side binding store prevents duplicate acceptance.

### Revocation
```
Authority publishes signed epoch → Platforms receive webhook → Verifiers apply staleness rules
```
Deterministic: same receipt + same revocation set = same verdict.
