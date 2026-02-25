# Contributing to VALORIN

## What's Open Source

The verifier ecosystem is open source under MIT license:
- Spec and conformance vectors
- Verifier core (Rust + WASM)
- Enforcement gates (GitHub Action, GitLab CI, Canvas LTI, D2L LTI)
- SDKs, CLI, and adapters

The authority layer (registry, revocation, transparency log, governance) is proprietary and hosted at api.valorin.dev.

## How to Contribute

### Conformance Vectors
Add test cases to `spec/v1/vectors/`. Every vector must include:
- `vector_id`: unique identifier
- `description`: what the vector tests
- `input`: complete input data
- `expected_verdict`: VALID, INVALID, or VALID_WITH_WARNINGS
- `expected_reason_codes`: ordered list of expected codes
- `expected_report_digest`: SHA-256 of expected canonical report

### Verifier Core
Contributions to the Rust verifier must:
- Pass all existing conformance vectors
- Not introduce nondeterminism (byte-identical outputs required)
- Not add network dependencies (verifier must work offline)
- Include tests

### Enforcement Gates
Gate contributions must:
- Use the verifier core (not custom verification logic)
- Implement anti-replay binding
- Log decisions with bundle_id/policy_hash/report_digest only (no receipt bodies)
- Handle all error cases defined in the spec

### Adapters
Adapter contributions must:
- Not store or transmit raw content
- Produce spec-compliant receipt payloads
- Include context binding for the target platform

## Code of Conduct

Be respectful. Be constructive. Focus on technical merit.

## Questions

Open an issue or reach out at support@valorinsystems.com.
