# AGENTS.md — fitness-app

**Audience:** AI agents and anyone implementing in this repository.

> Process: this repo follows Solo SDD — contracts in
> `specs/`, bets in `docs/pitches/`, decisions in `docs/adr/`, state in Linear.

---

## Product

- **What it is:** a training log whose differentiator is adapting the day's load to context — sleep, energy, and available time — instead of assuming the programmed routine.
- **Users:** one person, training in a gym with unreliable connectivity.
- **UI language:** Spanish. **Code, comments, tests, logs, commits and documentation are English** regardless.

---

## Structure

| Path | Role |
|---|---|
| `app/` | Next.js routes |
| `components/` | Shared UI |
| `features/` | Feature-scoped modules |
| `hooks/` | UI and data hooks |
| `lib/firebase/` | SDK setup, including offline persistence |
| `lib/schemas/` | Zod schemas — Firestore has no schema, so this is it |
| `lib/progression/` | The progression engine: pure TypeScript, no Firebase imports |
| `firestore.rules`, `firestore.indexes.json` | Authorization and indexes, deployed from CI |
| `specs/` | Contracts: `spec.md`, `design.md` per scope |
| `docs/pitches/` | Bets |
| `docs/adr/` | Architectural decisions |

---

## Invariants

Rules no spec, design or implementation may contradict without a new ADR.

1. **The training flow works with no network.** Writes go to Firestore's local cache first and the UI reads from it. A feature that requires connectivity to log a set does not ship.
2. **Document IDs are generated client-side.** A retried sync writes the same document, which makes it idempotent by construction. Never rely on server-generated IDs for anything written during a workout.
3. **`lib/progression/` imports nothing from Firebase.** It is pure functions over plain data, and it is tested to 100%. A bug there silently ruins months of training.
4. **Security Rules are tested** against the emulator, including cross-user access. A rule that is wrong does not fail — it silently allows.
5. **Every document is validated against its Zod schema before writing.** Firestore accepts anything; the schema is the only thing standing between a typo and a corrupt history.
6. **AI never modifies a session.** It proposes; the user taps; what was applied is recorded on the workout document.
7. **Nothing requires the Blaze plan.** No Cloud Functions, no Cloud Storage. If a feature needs either, it needs an ADR first.

---

## Toolchain

| Action | Command |
|---|---|
| Install | `npm ci` |
| Dev | `npm run dev` |
| Test | `npm test` |
| Test + coverage | `npm run test:coverage` |
| Rules tests | `npm run test:rules` (Firebase emulator) |
| Lint | `npm run lint` |
| Type-check | `npx tsc --noEmit` |
| Build (with gate) | `npm run build:ci` |
| Emulators | `npx firebase emulators:start` |

**Coverage threshold:** 90% global, 100% in `lib/progression/`, enforced by CI.

---

## Branching

Defaults from Solo SDD's `conventions/git.md`: trunk-based, one branch per scope,
`<prefix>/<ISSUE-ID>-<slug>`, squash to `main`, CI as the merge gate. No required
approvals while this is a one-maintainer repository.

---

## Conventions

- **Language:** English in code, comments, test names, error messages, logs and commits. User-facing strings are Spanish and live in the UI layer only.
- **Commits:** Conventional Commits, imperative.
- **Types:** strict; no `any` without a documented reason.
- **Tests:** colocated (`__tests__/` or `*.test.ts`).
- **Secrets:** never committed. `.env.local` is git-ignored; `.env.example` lists keys with empty values.
- **Test in airplane mode** anything that touches the workout flow.
