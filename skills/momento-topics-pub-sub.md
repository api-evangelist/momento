---
name: Publish and subscribe on a Topic
description: Send messages to a Momento Topic and consume them as a server-streamed subscription, handling discontinuities and heartbeats.
api: grpc/momento-cachepubsub.proto
operations: [Publish, Subscribe]
auth: authentication/momento-authentication.yml
errors: errors/momento-error-codes.yml
---

# Publish and subscribe on a Topic

Momento Topics is a best-effort multicast pub/sub bus. Topics live on a cache, so you
address them by `cache_name` + `topic`.

## Steps
1. **Publish** (`Publish`) — send a `_TopicValue` (either `text` or `binary`) to `cache_name`/`topic`. Publish returns `Ok` once accepted; it does not wait for subscribers and is **not** retryable (annotated `NotRetryable`).
2. **Subscribe** (`Subscribe`) — open a server-streaming subscription. You receive a stream of `_SubscriptionItem`s whose `kind` is one of:
   - `item` — a `_TopicItem` with `topic_sequence_number`, the `value`, and `publisher_id`.
   - `discontinuity` — Momento detected skipped/reordered messages; log it and continue.
   - `heartbeat` — a keepalive; ignore it (do not count it as a message).
3. **Resume** — to reconnect after a drop, pass `resume_at_topic_sequence_number` and the last `sequence_page` you saw.

## Rules
- Treat delivery as best-effort: never assume exactly-once. Deduplicate on `topic_sequence_number` if your app needs it.
- Do not retry `Publish` blindly — it is not idempotent at the bus level.
- For external delivery (Lambda/Slack/EventBridge) use Webhooks instead of a long-lived subscription — see `asyncapi/momento-webhooks.yml`.
