# Undrift — Focus & Calm
 
A consumer wellness iOS app for people with busy, restless minds — a daily companion for nervous-system regulation, focus, and calm. Built and shipped solo, live on the App Store.
 
**Status:** Live in production (App Store) · Solo developer & product owner
**Stack:** React Native · Expo · TypeScript (strict) · Zustand · Supabase · RevenueCat · PostHog
 
> This repository is a public write-up of how Undrift was built and the decisions behind it. The product source is private. The goal here is to document the architecture, the product thinking, and the AI-assisted development system that let one person ship and operate a production app.
 
---
 
## What it is
 
Undrift helps people who feel scattered or overwhelmed shift their state — calm down, focus, or re-regulate — without the friction that makes most wellness apps get abandoned. It is deliberately not "another meditation app." The core bet is **vertical depth for an underserved audience and retention over content breadth**, rather than competing on library size.
 
The experience spans a spectrum — calm → focus → energise → emotional reset — combining:
 
- **Emotional Reset** — a guided state-shift built on the ISO principle (match the current mood, bridge, then resolve toward calm).
- **Functional music + immersive soundscapes** — audio tuned for focus and arousal regulation rather than generic ambience.
- **Guided focus sessions** — narrated sessions supporting attention and follow-through.
- **Brain games + calming puzzles** — low-stakes cognitive activation and a sensory-settle alternative to doomscrolling.
---
 
## My role
 
Solo across the entire product: strategy, market positioning, UX, full-stack build, release engineering, analytics, and monetization. There was no team to hand work to — every decision below is one I owned end to end.
 
---
 
## Product decisions (the reasoning, not just the features)
 
The decisions matter more than the feature list. A few that show how the product was steered:
 
- **A single north-star metric.** The product is instrumented around `paywall_viewed → paywall_converted` as the primary success signal, with the broader funnel (onboarding → paywall → session) captured in analytics. Decisions are gated on data rather than opinion.
- **Metric-free calming experiences by design.** The calming games intentionally have no timer, no score, and no fail state. For an audience prone to overwhelm and self-criticism, performance framing is actively harmful — so "difficulty" is expressed as gentle progression (e.g. tier or anchor count), never as grid complexity or a clock. This was a deliberate "design *against* the obvious pattern" call.
- **Audience- and market-aware copy as a system, not a vibe.** Vocabulary and readability are stratified by surface and market — plain, B1-level English for non-native-English international markets, soft and outcome-led language in-app, and verb-outcome phrasing instead of quantity framing ("shift your state," not "162 tracks"). This is codified as a rule in a copy spec, not decided case by case.
- **Scope discipline.** Feature ideas are deferred to a backlog rather than built reactively, and architectural abstractions wait for a "rule of three" — a shared framework is only extracted once a third real use case justifies it. A browsing-heavy home layout was rejected because it rewarded aimless exploration and added friction for the exact users least served by it.
- **Tiered monetization wired to the product.** A subscription model (free tier + Pro) with real gating — a subset of content is free, the rest is Pro, enforced both in the UI and with a defensive backstop in the store layer so gating can't be trivially bypassed.
---
 
## Technical architecture
 
**Client** — React Native + Expo (managed), Expo Router for navigation, Zustand for state, TypeScript in strict mode. Feature-module structure: stores and components co-located by feature.
 
**Backend** — Supabase (Postgres + Auth). Row-level security on all tables, a `handle_new_user` trigger for profile provisioning, foreign-key cascade for clean deletes, and a `delete_own_account` RPC (App Store account-deletion compliance). Seeded content catalog (guided sessions, soundscapes, programs) served from the database.
 
**Auth** — Sign in with Apple in production via `expo-apple-authentication`, with route protection and persisted sessions. RevenueCat identity is re-associated to the authenticated user after sign-in so entitlements follow the account.
 
**Audio** — Multi-lane playback (music + environment + guided narration) with background-audio entitlement so playback continues when the screen is locked. Audio assets are served from a Cloudflare R2 CDN.
 
**Monetization** — RevenueCat for in-app purchases; subscription products verified purchasing on-device through StoreKit (multiple billing periods), with entitlement state refreshed on app foreground so access stays correct after expiry.
 
**Analytics** — PostHog (React Native SDK), 20+ events instrumented across the core funnel, EU-hosted, no PII.
 
---
 
## The AI-assisted development system
 
This is the part I'd most want a reader to understand, because it's *how* one person ships and maintains a production app at this scope.
 
Undrift isn't "I used an AI to write some code." It's a deliberate workflow designed to get small-team output from one person **without** sacrificing safety or coherence:
 
- **Strategy and implementation are separated.** Product strategy, specs, and design decisions happen in one context; implementation happens in another. The thinking is never entangled with the typing.
- **Work-package-gated execution.** No implementation starts without a written work-package spec. The agent doing the implementation receives a spec file, not a vague instruction — which keeps changes scoped, reviewable, and reversible.
- **Human review between phases.** Work is sequenced into phases with a review gate between them, so quality is checked at boundaries rather than discovered at the end.
- **Decisions are recorded, not remembered.** Non-obvious constraints live in Architecture Decision Records (30+ to date), so a constraint that bit once doesn't bite again. The codebase has a "read the ADR before touching the subsystem" rule.
The result is a system where velocity comes from leverage and discipline, not from cutting corners.
 
---
 
## Engineering discipline
 
- **Architecture Decision Records (30+).** Every load-bearing, non-obvious decision is documented with its rationale — from derived-state patterns to audio-lane behavior to release-pipeline rules.
- **Automated test suite.** 433 tests across 39 suites — covering store logic, session/streak/XP rules, and idempotent completion handling.
- **Release engineering.** EAS build pipeline with three profiles (dev / preview / production), dynamic per-variant config, and over-the-air updates via `expo-updates`. A strict OTA-safety rule governs it: any native-package change forces a full production rebuild before an OTA push — pure-JS changes only ship over-the-air.
- **A real post-mortem.** An early release shipped a stale JS bundle because a production binary was archived locally through Xcode instead of the EAS pipeline — meaning OTA updates couldn't reach those devices. The fix was a corrective EAS-built release plus a permanent rule: **production builds only ever go through EAS.** Documented so it can't recur.
- **Performance work under real constraints.** A session-player optimization eliminated thermal throttling on A15 devices by moving the timer to a shared-value/wall-clock model, consolidating animated layers, and throttling drag-time state commits — cutting re-renders during playback by roughly an order of magnitude with no perceptible change to the user.
---
 
## Roadmap (designed / in progress, not yet shipped)
 
These are specced or in progress, not live:
 
- Additional brain-training games (specced, build pending).
- An additional calming game and a shared gallery-game framework (the framework extraction is intentionally deferred until a third use case justifies it).
- An insights/personalization layer (in progress).
- A segmented paywall experiment (single-plan vs. tiered) by user intent — designed, not yet running.
- Audio caching and runtime perceptual leveling improvements.
---
 
## What this project demonstrates
 
- **0→1 product ownership** — concept to live App Store product, solo.
- **Product judgment** — a clear strategic bet, a single success metric, deliberate UX decisions, and the discipline to say no.
- **Technical execution** — full-stack mobile, a real backend with auth and security, working monetization, and release engineering.
- **An AI-assisted development system** — leverage with discipline, not shortcuts.
---
 
*Built and shipped by Ng Chee Woei. App available on the iOS App Store.*
