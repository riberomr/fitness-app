# ADR-0002: Embed sets in the workout document and denormalize per-exercise stats

**Date:** 2026-09-03
**Status:** Accepted

---

## Context

Firestore charges and counts **per document read**, and has no joins. The two access patterns that matter here are:

1. **During a workout**, for each exercise, answer "how did this go last time?" so the next set can be pre-filled. This runs several times a session, must be instant, and must work offline.
2. **Afterwards**, show the history of a session and the strength trend of an exercise over months.

A normalized relational instinct would give every set its own document. A 40-set session then costs 40 reads every time the history is opened.

## Decision

**Sets are embedded as an array inside the workout document.**

```
users/{uid}/workouts/{workoutId}
  { startedAt, finishedAt, routineId, adaptation, sets: [ {...}, ... ] }
```

**Per-exercise state is maintained in a denormalized document**, written in the same batch that closes a session.

```
users/{uid}/exerciseStats/{exerciseId}
  { lastSession, bestE1rm, currentTarget, sessionsWithoutProgress, totalSets }
```

**Document IDs are generated on the client**, which makes a retried sync idempotent: writing the same workout three times writes the same document three times, which is the same as writing it once.

## Alternatives considered

### One document per set

The normalized shape. Rejected: 40 reads to open one session, no natural transaction boundary for "the session is finished", and a much larger write count against the daily quota. The only thing it buys is querying sets across sessions directly — which is answered better by `exerciseStats` anyway.

### A subcollection of sets under each workout

A middle ground; still one read per set, plus an extra query per workout. Same objection, more indirection.

### Compute `exerciseStats` on read instead of maintaining it

Rejected. It would mean fetching and scanning a range of workouts every time the progression engine wants the last session — the operation that has to be instant and offline. Paying a little on write to make the hot read a single cached document is the right trade.

## Consequences

**Good**

- Opening a session is one read. Pre-filling the next set's weight is one cached document read, offline, instantly.
- The workout is a natural transactional unit: one document, written and read as a whole.
- Idempotent synchronization without deduplication code.

**Bad**

- `exerciseStats` is derived data that can drift from the workouts it summarizes. Mitigation: it is only ever written in the batch that closes a session, and a rebuild function must exist from the start to recompute it from history.
- "All sets of bench press in the last year" is not a direct query — it means fetching workouts in a range and filtering client-side. Acceptable: roughly 150 sessions a year for one user, already in the local cache.
- The 1 MiB document limit is a real ceiling. A 60-set session is around 15 KB, so there is no practical risk, but a future feature storing per-rep data would need to revisit this.

## Revisit when

A workout document approaches a few hundred KB, or a reporting need appears that genuinely requires querying sets across sessions server-side.
