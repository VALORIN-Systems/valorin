# Enforcement Gates

Enforcement gates convert verification into workflow control. No valid receipt = no merge, no submission, no execution.

## GitHub Action

Block pull request merges without valid receipts.

```yaml
- uses: valorin/verify-action@v1
```

## GitLab CI

Block merge requests without valid receipts.

```yaml
include: 'https://valorin.dev/ci/gitlab-verify.yml'
```

## Canvas LTI

Receipt-bound academic submissions via LTI 1.3 Deep Linking. Assignment configuration binds a policy hash. Submissions require a valid receipt. Replay across assignments is deterministically rejected.

## D2L Brightspace LTI

Same semantics as Canvas. Cross-platform verification produces identical results.

## Status

🚧 In development
