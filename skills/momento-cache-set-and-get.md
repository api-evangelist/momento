---
name: Cache a value and read it back
description: Store a key/value item in a Momento cache with a TTL, read it, atomically increment a counter, and delete it.
api: grpc/momento-cacheclient.proto
operations: [Set, Get, Delete, Increment]
auth: authentication/momento-authentication.yml
conventions: conventions/momento-conventions.yml
errors: errors/momento-error-codes.yml
---

# Cache a value and read it back

Use this skill to do basic key/value caching against a Momento cache. Momento Cache is
ephemeral: every write carries a TTL and expired items disappear automatically.

## Prerequisites
- A Momento API key (mint one in the Momento Console at https://console.gomomento.com/ or via the CLI).
- A cache. Create one with the control plane (`CreateCache`) or the CLI: `momento cache create my-cache`.
- Pass the API key as the `Authorization` header (HTTP) or `authorization` gRPC metadata.

## Steps
1. **Set** (`Set`) — write a key with a value and a TTL (`ttl_milliseconds` over gRPC, `ttl_seconds` over the HTTP API). Success returns no body.
2. **Get** (`Get`) — read the key. A hit returns the value; a miss is a distinct, non-error outcome (HTTP `404`, or a `Miss` result over gRPC) — always branch on hit vs. miss rather than treating a miss as failure.
3. **Increment** (`Increment`) — for counter use cases, atomically add a delta to a numeric value instead of read-modify-write.
4. **Delete** (`Delete`) — remove the key when done.

## Rules
- Always supply a TTL on writes; there is no "no-expiry" cache item.
- For optimistic concurrency use the conditional variants (`SetIf`, `SetIfHash`) rather than a client Idempotency-Key — Momento has no idempotency-key header, and writes are idempotent by key.
- On `RESOURCE_EXHAUSTED` / `429` (LimitExceededError) back off and retry; on `UNAVAILABLE`/`5xx` retry with jitter; do not retry `INVALID_ARGUMENT`/`UNAUTHENTICATED`.
