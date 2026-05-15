# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository status

**Docs-only right now.** No code, no build tooling, no tests. The only contents are specs in [docs/](docs/) that define a two-step lead-generation funnel for a newly licensed Orlando real estate agent targeting first-time buyers. When code lands, it will be an **Astro** site that posts to a **Make.com** webhook (no custom backend) — that stack assumption is baked into every spec.

## What this funnel is

Two lead magnets, gated entry, segmented by readiness. Both terminate at the same offer: a 30-minute **Buyer Strategy Session (BSS)**.

- **LM1** — *The Orlando First-Time Buyer Readiness Score* — 10-question scorecard, scored, segments into 3 tiers.
- **LM2** — *The 9-Step First Home Roadmap* — long-form process map, only promoted to Tier B from LM1 and from one standalone landing page.

Tiers (used interchangeably with internal names in code):
- **Tier A** = `READY_NOW` (display score 75–100) → CTA is the BSS
- **Tier B** = `NINETY_DAY` (45–74) → CTA is LM2
- **Tier C** = `FOUNDATION` (0–44) → CTA is nurture only; **never** pitch BSS here

## Document map and precedence

Read [docs/README.md](docs/README.md) first. Files are numbered for read order. The precedence rule when files seem to disagree:

- **Product specs** ([02-readiness-filter-spec.md](docs/02-readiness-filter-spec.md), [05-process-map-spec.md](docs/05-process-map-spec.md)) win for **behavior** — routes, scoring, payloads, overrides.
- **Content drafts** ([03-readiness-filter-content.md](docs/03-readiness-filter-content.md), [06-process-map-content.md](docs/06-process-map-content.md)) win for **copy**.
- **Email files** ([04-readiness-filter-emails.md](docs/04-readiness-filter-emails.md), [07-process-map-emails.md](docs/07-process-map-emails.md)) own transactional + nurture sequences.
- [08-implementation-roadmap.md](docs/08-implementation-roadmap.md) is the phased build plan and the source of canonical scoring test cases.

## Architecture you need in your head before editing

The funnel has **one webhook**. Both LM1 and LM2 forms POST the same Make.com endpoint, distinguished by a `magnet` field (`"lm1"` or `"lm2"`). Make.com owns: contact storage, email sending, PDF attachment, sequence enrollment/pausing, and the Google Sheet fallback log. The Astro app owns: rendering, scoring, signed result tokens, and analytics events.

Two non-obvious cross-magnet rules Make.com must enforce — easy to miss when editing either spec in isolation:

1. When `source = "lm1_tier_b"` arrives on the LM2 webhook, **pause** the existing LM1 Tier B nurture so the contact does not get a double email storm.
2. Tier C contacts are **never** pitched the BSS from any sequence; their CTA is education + bi-weekly nurture indefinitely.

The result page (`/readiness/result`) is **fully static**, driven by a short-lived HMAC-signed token in the query string (`{n, t, s, exp}`). No DB lookup. This keeps Make.com out of the render path and lets the page survive without a backend.

## Configuration principle (load-bearing)

Any value flagged "calibrate against real data" in the specs — tier thresholds, per-question point values, override actions, token expiry, copy strings — **must live in a single config module** (the specs suggest `src/config/readiness.ts` or a JSON variant). Inline-hardcoding any of these will require a refactor in Phase 4 when thresholds get tuned against real submission distributions. See the "Configuration" section of [02-readiness-filter-spec.md](docs/02-readiness-filter-spec.md) for the full list.

Structural constants (e.g., "there are 10 questions") may be inline. The test: if the spec has a "this is our current guess" comment next to a value, it belongs in config.

## Locked content — copy verbatim

These are explicitly locked in the specs. Don't paraphrase them on landing pages, ads, DMs, or in nav copy:

- **"The Orlando First-Time Buyer Readiness Score"** + subhead *"Know in 7 Minutes If You're 30, 90, or 180 Days Away From Your First Home."*
- **"The 9-Step First Home Roadmap"** + subhead *"Exactly What Happens Between 'I Think I'm Ready' and Keys in Your Hand in Orlando."*

Also locked: the **stable enum keys** for each scorecard answer (e.g., `q1_credit_range: "680_739"`). Make.com routes on these — renaming them breaks the email scenario. Human-readable labels in the UI can be re-copy-edited freely; the underlying enum keys cannot.

## Working voice and the Grand Slam Offer test

All user-facing copy — landing pages, quiz screens, result pages, transactional and nurture emails, CTAs — is written in the agent's voice: **direct, warm, educational, never salesy**. Two Hormozi tests every piece of copy must pass before it ships:

- *"Would the prospect feel stupid to say no?"* — the offer must stack enough value that declining feels irrational.
- *"Does this feel like they're getting a deal, not being sold to?"* — lead with usefulness, never with a pitch.

Every offer in the funnel — including LM1, LM2, and every CTA inside them — is a **Grand Slam Offer** evaluated against Hormozi's value equation:

```
Value = (Dream Outcome × Perceived Likelihood of Achievement) / (Time Delay × Effort & Sacrifice)
```

[01-strategy-and-funnel.md](docs/01-strategy-and-funnel.md) already shows this applied to LM1 and LM2. When editing headlines, subject lines, or CTA copy, optimize for **all four** drivers — don't silently trade one for another. Lead magnets are mini-offers: never propose a vague freebie ("Free Buyer's Guide"); always name the specific result and the time-to-result (the LM1 / LM2 locked titles are the template).

## Market context to bake into examples and copy

Default these unless the user tells you otherwise — generic "Florida" examples are a smell.

- **Geography**: Orlando metro, with depth along the **I-4 corridor between Sanford and Downtown Orlando**, spanning both **Seminole and Orange counties**. On-corridor cities: Sanford, Lake Mary, Longwood, Altamonte Springs, Maitland, Winter Park, Downtown Orlando. East of I-4 along the corridor: Oviedo, Winter Springs, Casselberry. West of I-4 along the corridor: Apopka, Eatonville, College Park. Snapshots, gotchas, and price examples should name these specifically — do **not** narrow to Seminole-only or widen to "all of Orlando metro" (Lake Nona, Dr. Phillips, Kissimmee, Clermont are out of scope unless the user names them).
- **Primary personas**: first-time buyers, families relocating from Puerto Rico or other states, move-up buyers leaving older neighborhoods.
- **Agent's edge**: analytical / data-fluent, bilingual (English / Spanish, Florida Latino register — Cuban or Puerto Rican Spanish, not Castilian), deep local knowledge. Copy should sound like a diagnostician, not a salesperson.

## Scoring engine specifics (when you build it)

- Raw max is 89; display is normalized to 100 via `round((raw / 89) * 100)`.
- Tier overrides apply **after** the threshold lookup, in the order listed in [02-readiness-filter-spec.md](docs/02-readiness-filter-spec.md). Once a tier has been demoted, do **not** re-promote it.
- The 6 canonical test cases the scoring engine must pass are in [08-implementation-roadmap.md:97-105](docs/08-implementation-roadmap.md#L97-L105). Each tier and each override has at least one canonical case.
- Implement the scorer as a pure TypeScript function that reads config; no UI dependencies. This makes it unit-testable without Astro.

## Advisory modes — when the user isn't asking you to write code

The user is the agent themselves (a newly licensed Orlando REALTOR with an iOS / Ruby / Rails background) and will frequently ask growth, offer, or content questions that have nothing to do with the Astro build — e.g., *"How do I script a DM to my college roommate?"*, *"What should I post Monday?"*, *"How do I package a relocation service?"*. When that happens, respond in **one** of these five structured modes (from the project's source brief):

1. **Offer Builder** — service/package questions → Dream Outcome + 3–5 bonuses + plain-English offer + risk-reversing guarantee + CTA.
2. **Lead Magnet Designer** — freebie/opt-in questions → title that names a specific result + one-sentence promise + 3–5 named deliverables + which warm-outreach method delivers it.
3. **Warm Outreach Scripts** — "how do I message X?" → exact message for the channel (text, DM, email, voice note) with reactivation opener → give → natural transition → open-loop close. **Never** pitch-first.
4. **Content Plan** — "what do I post?" → identify type (**What I Know** / **Results** / **Story** / **Engagement**), write the post ready to paste, attach a low-friction CTA that points at a lead magnet, label the platform and format.
5. **Strategy Session** — situation diagnosis → name the bottleneck (offer clarity, lead volume, or conversion), prescribe the single highest-leverage action this week, give a daily plan with specific numbers.

Default cadence to assume unless told otherwise: **5–10 personal warm messages/day, 3 posts/week minimum** until the pipeline is full. Stay on platforms the agent already uses (Facebook, Instagram, YouTube) — never propose starting a new one. Content must be one of the four approved types; "inspirational without specific usefulness" is not a type.

Every advisory output must be specific enough to act on **today** — exact words, exact numbers, exact next step. Vague advice fails the brief.

## Hard "do not build" / "do not recommend" rules

Strict Hormozi-rules (*$100M Offers* / *$100M Leads*). These bind **both** code/spec work and advisory output — do not introduce them even if asked offhandedly:

- No paid ads or boosted posts
- No cold outreach, lead-list purchases, or scraped contacts
- No pop-up "Get my free home valuation" widgets
- No generic "First-Time Buyer's Guide" PDFs with no diagnostic specificity
- No third lead magnet — exactly LM1 and LM2 by design; a third fragments attention
- LM2 is **only** promoted from the LM1 Tier B result and the `/roadmap` standalone landing page; do not add it to homepage nav, bios, or Tier A/C result pages
- No tactic that isn't documented in *$100M Offers* or *$100M Leads*. When in doubt, run the Hormozi test: *"Does this make the prospect feel like they'd be stupid to say no?"*

## What is intentionally out of scope

- Backend/data-model code (Make.com is the backend)
- Self-hosted CRM, custom calendar tool, buyer portal, or native mobile apps
- Spanish translation drafts (deferred to Phase 3; flagged in the roadmap but not authored)
- The BSS offer doc itself (next deliverable after this funnel ships)

## Build / test commands

None yet — there is no Astro project scaffolded. When Phase 1 starts (see [08-implementation-roadmap.md](docs/08-implementation-roadmap.md)), the expected commands will be standard Astro (`npm run dev`, `npm run build`, plus a unit-test runner for the scoring engine). Update this section once `package.json` exists.
