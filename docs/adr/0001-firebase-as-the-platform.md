# ADR-0001: Use Firebase as the whole backend platform

**Date:** 2026-09-03
**Status:** Accepted

---

## Context

Two side projects were planned together — this training log and an expense tracker — with a hard constraint of staying on free tiers, and a secondary goal of learning both major backend-as-a-service platforms rather than one.

The hardest problem in this application is not the data model or the progression maths. It is that **gyms have basements, thick walls and no signal**, and losing a logged set to a dead connection is unacceptable. Everything else is secondary to that.

Relevant facts as of this date:

- The Firestore SDK ships **offline persistence and automatic synchronization**: writes hit a local cache first and reconcile when connectivity returns, with no synchronization code to write.
- Firestore Spark tier: 50k reads, 20k writes and 20k deletes per day, 1 GiB stored. **It does not pause for inactivity.**
- Firebase **Cloud Storage requires a linked billing account since 3 February 2026**, and **Cloud Functions require the Blaze plan** — both mean a credit card.
- **Firebase AI Logic works on the Spark plan** with the Gemini Developer API free tier, calling models from the client without exposing an API key.
- Supabase free tier pauses a project after seven days of inactivity and allows only two active projects.

## Decision

This application uses Firebase for everything: Auth, Firestore with local persistence, Security Rules, and AI Logic for the contextual adaptation. The companion expense tracker uses Supabase.

The architecture deliberately avoids anything requiring the Blaze plan. This is recorded as an invariant in `AGENTS.md`: no Cloud Functions, no Cloud Storage without a new ADR.

## Alternatives considered

### Supabase for both applications

Rejected. It would mean building offline support by hand: IndexedDB, a write queue, retry with backoff, and idempotency keys to stop a flaky connection producing duplicate sets. That is weeks of work on the highest-risk part of the product, replacing something the Firestore SDK already does correctly.

### Firebase for both applications

Rejected on the expense tracker, not on this one. That application is aggregation-heavy and needs receipt storage plus scheduled jobs — all three either cost money or fight the tool on Firestore.

### A native mobile application instead of a PWA

It would unlock Health Connect and HealthKit, which a PWA cannot reach. Rejected for now as a much larger commitment for a benefit that manual entry approximates in five seconds a day. Deferred rather than dismissed — see the wearables question in the product spec.

## Consequences

**Good**

- The single hardest requirement — logging works with no signal — is met by the platform rather than by code I have to maintain.
- Client-generated document IDs make synchronization idempotent by construction: a retried write targets the same document.
- No scheduled jobs are needed, and Firestore does not pause, so there is no keep-alive to maintain.
- Gemini is reachable from the client without a server or a leaked key.

**Bad**

- **No progress photos in the MVP**, because Storage needs a card. Numeric measurements cover the same purpose for now.
- Firestore has no schema, so validation has to be enforced in code. Zod schemas at every write are an invariant, not a preference.
- Reads are billed and counted per document, which forces modelling for reads — sets embedded in the workout document, and a denormalized `exerciseStats` document per exercise.
- Aggregations across a date range are client-side work, not a query.

**Neutral**

- Two platforms across two projects means context-switching, which is the cost of the learning goal.

## Revisit when

Progress photos or a wearable integration become genuinely necessary, or a feature appears that cannot work without server-side compute. Enabling Blaze with a zero-dollar budget alert is the smaller step at that point; migrating platforms is not.
