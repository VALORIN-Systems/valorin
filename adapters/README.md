# Receipt Adapters

Receipt adapters add VALORIN receipt emission to existing tools. When a tool has an adapter, it automatically produces receipts that can be verified by enforcement gates.

## How Adapters Work

```
Developer uses tool (Copilot, Cursor, etc.)
    → Adapter wraps tool output with receipt metadata
        → Receipt issued with policy binding + context binding
            → Enforcement gate verifies receipt
```

## Planned Adapters

| Tool | Type | Status |
|---|---|---|
| GitHub Copilot | AI coding assistant | Planned |
| Cursor | AI coding assistant | Planned |
| OpenAI API | AI API | Planned |
| Anthropic API | AI API | Planned |
| Jest | Testing framework | Planned |
| pytest | Testing framework | Planned |

## Building an Adapter

Adapters implement the `ReceiptEmitter` interface:

1. Capture tool metadata (version, configuration, execution context)
2. Produce commitments (hashed digests, bucketed counts/durations — no raw content)
3. Bind to context (repo/PR/SHA for CI, course/assignment/run for LMS)
4. Request receipt issuance from authority or local signer

See `sdk/` for Rust and TypeScript SDKs.

## Status

🚧 In development
