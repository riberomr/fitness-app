# fitness-app

Training log that **adapts the session to the day you are actually having** — sleep, energy, and the forty minutes you really have — instead of assuming the plan on paper.

> **Status:** pre-development. The contract for the first scope is being written.
> Nothing here runs yet.

## The idea

Logging apps are good at recording sets and blind to context. The programmed routine assumes eight hours of sleep and ninety free minutes; the reality is five hours of sleep and forty minutes. The differentiator here is the biometric and situational context applied to the day's load.

Deliberately **not** an AI coach that chats. Nobody wants a conversation with their phone between sets. Adaptation is one screen, ten seconds, two buttons: accept or adjust.

## Principles

1. **Logging speed is the product.** Two taps, or one spoken sentence, per set.
2. **Offline-first, not offline eventually.** Gyms have basements and concrete. Losing a set to a dead connection is a critical bug.
3. **Progression is deterministic; AI is the contextual exception.** Progressive overload is arithmetic with rules, not a model's opinion.
4. **AI proposes, never modifies.** Every adaptation needs an explicit tap, and what was applied is recorded — otherwise the training history lies.
5. **Free tier by default.** Any recurring cost is justified in writing before it is incurred.
6. **The data belongs to the user.** Full export from day one.

## Stack

Next.js 16 (App Router) · React 19 · TypeScript · Tailwind · PWA with Wake Lock · Firebase (Auth, Firestore with offline persistence, Security Rules, AI Logic) · deployed on Vercel.

Decisions that were expensive to make are recorded in [`docs/adr/`](docs/adr/).

## Process

This repository follows Solo SDD: the bet lives in `docs/pitches/`, the contract and design in `specs/`, architectural decisions in `docs/adr/`, and status in Linear. See [`AGENTS.md`](AGENTS.md) for conventions.
