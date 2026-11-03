# Generated component errors and recovery

Use only structured MCP fields. Never request server paths, stack traces, bundles, tokens, or logs.

| Error/diagnostic | Automatic action |
| --- | --- |
| `COMPONENT_PREVIEW_INPUT_INVALID` | Fix unknown/missing/type-invalid Props, Schema dialect, JSON size/safety, or missing preview variables using `path`, `prop`, and `expectedType` when present. |
| `COMPONENT_PREVIEW_COMPILE_FAILED` | Read `phase`, module/export, allowed exports, location, and limits; edit and resubmit. |
| `import-not-allowed` | Use `moduleName`, `requestedExport`, and `allowedExports`; choose a listed API or supported alternative. |
| `syntax-error`, `default-export` | Fix the reported location/shape without adding unrelated code. |
| `source-too-large`, `too-complex`, `bundle-too-large`, `sandbox-bundle-too-large` | Reduce the candidate using `limit`/`actual`; do not split it into an app or bypass the sandbox. |
| `sandbox-bundle-error` | Re-read capabilities once. If the same supported source still fails, report a platform runtime fault; do not rewrite valid TSX blindly. |
| `COMPONENT_PREVIEW_BASE_CONFLICT` | Re-read current source detail and regenerate against the new version/hash. |
| `COMPONENT_PREVIEW_OWNER_NOT_READY` | Ask the user to open the authenticated review page; do not forge READY or approval. |
| owner/agent `RUNTIME_ERROR` | The error is terminal for that preview source hash so a late READY cannot reopen approval. Fix the cause and create a new preview; do not reuse the failed session. |
| `COMPONENT_PREVIEW_TOKEN_INVALID` / expired URL | Explicitly refresh once, then use only the new URL. |
| `COMPONENT_PREVIEW_NOT_APPROVED` | Poll status or wait for the user; do not finalize. |
| `COMPONENT_PREVIEW_DECISION_CONFLICT` | Read status and follow the recorded terminal decision. |
| missing scope / disabled writes | Ask the user to authorize or enable the required GrowthTrace capability. Do not route around it. |

Compilation retries continue while diagnostics are actionable and effective input changes. They do not count toward the three visual rounds. If the same code recurs with no meaningful input change, stop blind retries, refresh capabilities, and decide whether a supported rewrite exists. Report to the user only for an authorization/approval/product choice, a confirmed distribution mismatch, or a verified unsupported requirement with no equivalent supported implementation.
