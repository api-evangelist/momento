---
name: Maintain a leaderboard and read ranks
description: Upsert scored elements into a Momento Leaderboard, read a page of elements by rank, look up specific ids' ranks, and remove elements.
api: grpc/momento-leaderboard.proto
operations: [UpsertElements, GetByRank, GetRank, RemoveElements]
auth: authentication/momento-authentication.yml
errors: errors/momento-error-codes.yml
---

# Maintain a leaderboard and read ranks

A Momento Leaderboard is a durable sorted set: each element is a `uint64` id mapped to a
single single-precision float `score`. An id can appear only once.

## Steps
1. **UpsertElements** (`UpsertElements`) — insert or update up to 8192 elements in one call. Upsert is all-or-nothing (no partial success).
2. **GetByRank** (`GetByRank`) — fetch a range of elements by 0-based ordinal rank (up to 8192 per call); page with the offset for larger ranges.
3. **GetRank** (`GetRank`) — look up the current rank of specific ids (e.g. "where does player X stand?").
4. **RemoveElements** (`RemoveElements`) — remove up to 8192 elements by id.

## Rules
- Batch limit is 8192 elements per call for the multi-element APIs — chunk larger updates.
- Scores are IEEE-754 single-precision floats: integers above 16,777,216 (or below -16,777,216) are not all exactly representable; keep score magnitudes within that range for exact integer ranking.
- Use `GetByScore` (score range) when you want a window by value rather than by rank, and `DeleteLeaderboard` to stop incurring storage for a whole board.
