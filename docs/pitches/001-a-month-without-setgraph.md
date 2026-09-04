# Pitch: A month without Setgraph

**Date:** 2026-09-04
**Appetite:** Big — 10 sessions
**Status:** bettable
**Linear Project:** [Fitness · A month without Setgraph](https://linear.app/martin-ribero/project/fitness-a-month-without-setgraph-fae329a30986)

---

## 1. Problem

There is already a training log on this phone, and it works. Setgraph records a set between one rest and the next, one-handed, without thinking about it. Any replacement that is slower than that has already lost, no matter what else it does.

What Setgraph does not do is notice what kind of day it is. The routine on paper assumes eight hours of sleep, ninety free minutes and a body that is not wrecked. The actual Tuesday is five hours of sleep, forty minutes between obligations, and shoulders that still hurt from Sunday. Facing that, there are two bad options: train the prescribed session badly, or skip it. Both corrupt the record — the first logs numbers that misrepresent the effort, the second logs nothing at all.

That gap is the whole reason to build anything here. But it only becomes reachable **after** logging is at least as fast as what it replaces, because a training log nobody opens has no context to adapt.

## 2. Appetite

**Big — 10 sessions.**

This bet is the floor the differentiator stands on: the data model, the offline guarantee, and a logging screen that survives comparison with a shipped product. It gets one full cycle and not a second one. If logging is still slower than Setgraph after ten sessions, the answer is to cut scope inside this bet — not to move on to adaptation on top of a log that does not get used.

> This is a cap, not an estimate. If the solution does not fit in the appetite,
> the scope gets cut — the appetite does not move.

## 3. Solution

A training log that is **never the reason a set went unrecorded**: not the basement with no signal, not the two seconds spent hunting for last week's weight, not the phone locking mid-rest.

1. **The next set is already filled in.** Weight and reps come pre-filled from what happened last time in that exercise, read from a single document that is already in the local cache. Confirming is one tap; correcting is a tap and a number.
2. **The rest timer starts itself.** Logging the set starts the clock. Nobody presses start on a timer with a barbell in their hands.
3. **It works in a basement.** Every write lands in the local cache first and the screen never waits for the network. Airplane mode is a test case, not a degraded mode.
4. **Yesterday's numbers are visible.** Per-exercise history with the e1RM trend — enough to know whether the last two months went anywhere.
5. **The data leaves whenever it wants to.** Full JSON/CSV export from the first release. A log you cannot get out of is a log you are renting.

**Main flow:**

1. The session opens on today's routine day, with each exercise's target already computed from history.
2. A set is done. The screen shows the proposed weight and reps; one tap logs it, or the number is corrected first.
3. The rest timer runs on its own. The next set is queued with its own proposal.
4. Closing the session writes the session document and updates the per-exercise stats in the same batch — so the next session opens instantly and offline.
5. Any time later, the history screen shows how a given exercise has moved.

**The rule everything else bends around:** a set is recorded locally the instant it is confirmed, with no network in the path. Sync is the SDK's problem, not the user's.

## 4. Rabbit holes

| Risk | Decision |
|---|---|
| Offline sync is the classic place to lose data, and hand-rolling it is weeks of work | **Firestore's own persistence, and nothing on top of it** (ADR-0001). No second cache, no retry queue, no TanStack Query. Document IDs are generated on the client, so a retried write is the same write — idempotency comes from the structure, not from deduplication code (ADR-0002). |
| "Fast logging" is a goal nobody can fail | Held to a **counted interaction budget** — a set logged in ≤ 2 taps from the training screen — verified as an acceptance criterion, not as an impression. |
| Reading a session's history could cost a read per set | **Sets are embedded in the session document** (ADR-0002): a whole session is one read. The honest cost is that "every bench press set this year" is a client-side filter over a date range, which for one user is trivial and already cached. |
| The progression proposal could quietly become the full deterministic engine | This bet ships **the last-session lookup only** — what was done last time, in this exercise, pre-filled. Rule tables, double progression and deload detection are the next bet. The `exerciseStats` document is written here so the engine has somewhere to land. |
| Suggesting weights the gym does not have | `incrementKg` is per-exercise and per-equipment from the first schema. A proposal that rounds to a plate nobody owns is worse than no proposal. |
| Starting with an empty history kills the motivation to switch | **Resolved before the bet starts, by a spike** (scope 0). If Setgraph exports a usable file, importing it is in this bet; if it does not, the answer is known before the document shape is frozen rather than after. |

## 5. No-gos

- **Voice dictation.** Next bet. It is the second-fastest path to a logged set and it does not exist until the fastest one does.
- **Readiness capture and AI adaptation.** The differentiator, and deliberately not first: it has nothing to adapt until a month of real sessions exists. Bets 2 and 4.
- **The deterministic progression engine.** Rule tables, double progression, deload suggestions — bet 3. This bet pre-fills from last time, which is the part that saves the two seconds.
- **Weekly volume by muscle group, tonnage, anthropometry.** Read-side work on data that does not exist yet.
- **Progress photos.** Blocked by Firebase Storage requiring a card since February 2026 (decision D6). Numeric measurements do the job; revisit in bet 3.
- **Wearables.** Health Connect and HealthKit are not reachable from a PWA at all (decision D2). Manual entry when readiness arrives; anything else is bet 5 at the earliest.
- **A native application.** The whole platform choice assumes a PWA (ADR-0001); nothing here revisits it.

## 6. Scopes

| # | Scope | Covers | Blocked by | Linear |
|---|---|---|---|---|
| 0 | **Spike: does Setgraph export anything usable?** Decision D3 — resolve before the document shape is frozen | Rabbit hole 6 | — | [MAR-11](https://linear.app/martin-ribero/issue/MAR-11) |
| 1 | **Foundations.** Auth, the Firestore document model, Security Rules with cross-user tests, the emulator as the development environment. Demoable: sign in, create an exercise, see it | Main flow 1 | — | [MAR-13](https://linear.app/martin-ribero/issue/MAR-13) |
| 2 | **Catalogue and routines.** The global exercise catalogue, the user's own exercises with their real increments, and a routine as days of exercises with rep ranges, rest and priority | Main flow 1 | scope 1 | [MAR-14](https://linear.app/martin-ribero/issue/MAR-14) |
| 3 | **The set logger.** Training screen, ≤ 2 taps per set, pre-fill from `exerciseStats`, self-starting rest timer, session close writing stats in the same batch — verified in airplane mode | Solution 1–3, main flow 2–4 | scope 2 | [MAR-15](https://linear.app/martin-ribero/issue/MAR-15) |
| 4 | **History and export.** Per-exercise history with the e1RM trend, and full JSON/CSV export | Solution 4–5, main flow 5 | scope 3 | [MAR-16](https://linear.app/martin-ribero/issue/MAR-16) |

## 7. How this bet is judged

**One month of training without opening Setgraph.** Not "the features shipped" — the old app going unopened. If it gets opened, the cause is logging speed, and nothing else gets built until that is fixed.

---

## Enabling work outside the spine

The repository has no application in it. [MAR-12](https://linear.app/martin-ribero/issue/MAR-12) scaffolds one and its quality gate — no acceptance criterion of its own, so it is not a scope — and blocks scope 1. Its first task is an ADR on the framework: `AGENTS.md` still says Next.js, written before anyone checked whether it fits an offline-first application that is entirely behind authentication and talks straight to Firestore. finance-app made the same assumption and replaced it with Vite in its ADR-0006.
