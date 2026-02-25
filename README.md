# VALORIN

Deterministic receipt-based verification for governed workflows.

---

## What VALORIN Does

When code moves through a CI pipeline or a student submits AI-assisted work through an LMS, VALORIN produces a **cryptographic receipt** — a signed, immutable proof that the workflow executed under a specific policy with specific tools.

That receipt can be verified by anyone, offline, deterministically, without trusting a vendor.

## How It Works

```
Workflow executes under policy
    → Receipt issued (signed, content-addressed, immutable)
        → Gates enforce (no valid receipt = no merge / no submission)
            → Anyone can verify (deterministic, offline-capable)
```

### Verification Pipeline

```
Parse → Canonicalize (JCS) → Hash (SHA-256) → Signature (Ed25519)
  → Policy Eval → Revocation Eval → Registry Eval → Emit Report
```

Same inputs. Byte-identical output. Every time.

## Architecture

### Open Source — The Verifier Ecosystem

| Component | Description |
|---|---|
| `spec/v1/` | Receipt Verification Spec — wire format, semantics, conformance vectors |
| `verifier-core/` | Deterministic verification kernel (Rust) |
| `verifier-wasm/` | WASM build — runs in browsers, CI, embedded, anywhere |
| `gates/github-action/` | GitHub Action — block merges without valid receipts |
| `gates/gitlab-ci/` | GitLab CI template — equivalent enforcement |
| `gates/canvas-lti/` | Canvas LTI 1.3 — receipt-bound academic submissions |
| `gates/d2l-lti/` | D2L Brightspace LTI — same semantics |
| `sdk/` | Rust and TypeScript SDKs |
| `cli/` | `valorin verify receipt.json` |
| `adapters/` | Receipt adapters for existing tools |

### Authority Layer — Hosted by VALORIN

The verifier works fully offline. The authority layer provides:

- **Registry** — tool/version certification authority
- **Revocation** — signed epoch-based revocation sets
- **Transparency Log** — append-only audit of all authority actions
- **Governance** — multi-party approval for authority changes

Default authority: `api.valorin.dev`. Configurable to any endpoint or local files.

## Quick Start

### GitHub CI (3 lines)

```yaml
# .github/workflows/verify.yml
name: Receipt Verification
on: [pull_request]
jobs:
  verify:
    runs-on: ubuntu-latest
    steps:
      - uses: valorin/verify-action@v1
```

### CLI

```bash
valorin verify receipt.json
valorin verify receipt.json --trust-store local.json --revocations local.json
```

### TypeScript

```typescript
import { verify } from '@valorin/verify';

const report = await verify(bundle, { mode: 'POLICY_VERIFY', policy });
// report.verdict: 'VALID' | 'INVALID' | 'VALID_WITH_WARNINGS'
// report.reason_codes: deterministically ordered, stable semantics
```

## Cryptographic Properties

| Property | Specification |
|---|---|
| Canonicalization | RFC 8785 (JCS). Rejects duplicate keys and floats. |
| Signatures | Ed25519. Domain-separated: `VALORIN_ATTESTATION_V1 \|\| 0x00 \|\| canonical(payload)` |
| Hashing | SHA-256 for all content addressing |
| Revocation | Signed epoch-monotonic sets. Policy-selected staleness: fail-open or fail-closed. |
| Determinism | Identical inputs → byte-identical verification reports |

## Certification

Tools that produce receipts can be certified through the VALORIN registry.

| Tier | What It Proves |
|---|---|
| **Conformance** | Tool produces spec-compliant receipts |
| **Governance** | Tool meets audit trail and policy binding requirements |
| **Privacy** | Tool stores no raw content and uses pseudonymous bindings |

## What VALORIN Does Not Do

- No AI detection or authorship claims
- No proctoring, surveillance, or monitoring
- No raw content storage by default
- No claim that "certified" means "secure"
- No network required for verification

## Privacy Posture

Receipts store policy hashes, commitments (bucketed counts and durations), and context bindings. Receipts do **not** store prompts, outputs, essays, code, or any raw content by default.

This makes VALORIN deployable in privacy-sensitive environments — including universities — without triggering data sensitivity review.

## Project Status

**Active development.** Building toward v1.

- [ ] Spec v1 locked
- [ ] Verifier-core (Rust) — conformance vectors passing
- [ ] Verifier-core (WASM) — byte-equality proven
- [ ] GitHub Action enforcement gate
- [ ] GitLab CI enforcement gate
- [ ] Canvas LTI integration
- [ ] D2L LTI integration
- [ ] Authority layer (registry + revocation + transparency + governance)
- [ ] CLI
- [ ] Documentation site

## Spec

See [`spec/v1/`](spec/v1/) for the full Receipt Verification Spec including schemas, conformance vectors, reason code taxonomy, and cryptographic specification.

## License

Verifier, spec, gates, SDKs, CLI, adapters: **MIT License**

Authority layer: Proprietary (hosted at api.valorin.dev)
