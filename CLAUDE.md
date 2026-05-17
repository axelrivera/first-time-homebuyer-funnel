# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository status

**Docs-only right now.** No code, no build tooling. The only contents are specs in [docs/](docs/) that define a two-step lead-generation funnel for a newly licensed Orlando real estate agent targeting first-time buyers. When code lands, it will be an **Astro + Tailwind CSS** static site (vanilla JavaScript, no backend, no cookies, no environment variables) that POSTs to a **Make.com** webhook, which creates a Person/Lead in **Pipedrive**; **Pipedrive Campaigns** (with Workflow Automations) owns transactional and nurture email. That stack assumption is baked into every spec.

## What this funnel is

Two lead magnets, gated entry, segmented by readiness. Both terminate at the same offer: a 30-minute **Buyer Strategy Session (BSS)**.

- **LM1** — *The Orlando First-Time Buyer Readiness Score* — 10-question scorecard, scored, segments into 3 tiers.
- **LM2** — *The 9-Step First Home Roadmap* — long-form process map, only promoted to Tier B from LM1 and from one standalone landing page.

Tiers (used interchangeably with internal names in code):
- **Tier A** = `READY_NOW` (display score 75–100) → CTA is the BSS
- **Tier B** = `NINETY_DAY` (45–74) → CTA is LM2
- **Tier C** = `FOUNDATION` (0–44) → CTA is nurture only; **never** pitch BSS here

## Document map and precedence

This feature has three artifact trees, each with a different job. Don't collapse them.

### [docs/](docs/) — product specs (permanent)

The source of truth for *what this funnel should do and say*. Files are numbered for read order; start with [README.md](README.md).

- **Product specs** ([02-readiness-filter-spec.md](docs/02-readiness-filter-spec.md), [05-process-map-spec.md](docs/05-process-map-spec.md)) win for **behavior** — routes, scoring, payloads, overrides.
- **Content drafts** ([03-readiness-filter-content.md](docs/03-readiness-filter-content.md), [06-process-map-content.md](docs/06-process-map-content.md)) win for **copy**.
- **Email files** ([04-readiness-filter-emails.md](docs/04-readiness-filter-emails.md), [07-process-map-emails.md](docs/07-process-map-emails.md)) own transactional + nurture sequences.
- [08-implementation-roadmap.md](docs/08-implementation-roadmap.md) is the phased build plan and the source of canonical scoring examples.

### [requirements/](requirements/) — build tasks (active during build, archival after)

Sequential, self-contained Claude Code sessions with a Definition of Done per task. Read [requirements/README.md](requirements/README.md) for the overall order. Tasks don't duplicate spec content; they reference docs/ and defer to `EXISTING-SITE-NOTES.md` for project-specific patterns.

- [00-foundations/](requirements/00-foundations/) runs first; its audit task produces the notes file below.
- [01-lm1-readiness-filter/](requirements/01-lm1-readiness-filter/) (tasks 01–10) builds LM1 end to end.
- [02-lm2-process-map/](requirements/02-lm2-process-map/) (tasks 01–05) builds LM2; do not start until LM1 task 09 ships.

### `requirements/EXISTING-SITE-NOTES.md` — host-site bindings (stable reference)

Produced by [requirements/00-foundations/01-audit-existing-site.md](requirements/00-foundations/01-audit-existing-site.md). Documents the host Astro site's `BaseLayout`, analytics helper, Tailwind setup, folder conventions, and routing. Every downstream build task implicitly defers to it. Until it exists, **no LM1 or LM2 task is unblocked** — improvising patterns the host site already has is the exact failure mode the audit prevents.

### Precedence when sources disagree

1. **Spec / behavior / copy conflicts** → [docs/](docs/) wins. Build tasks describe intent; specs describe truth.
2. **"How does this host site do X?"** (layout props, analytics signature, styling, route conventions) → `EXISTING-SITE-NOTES.md` wins. Task files only describe funnel-specific intent; the notes file is the binding to the real codebase.
3. **"What do I build next?"** → [requirements/](requirements/) in numeric order, gated on `EXISTING-SITE-NOTES.md` existing for anything past 00-foundations.

## Architecture you need in your head before editing

```
[Astro static site] ──POST──▶ [Make.com webhook] ──▶ [Pipedrive Person/Lead]
       │                              │                         │
       │                              ├─▶ [Google Sheet log]    └─▶ [Pipedrive Workflow
       │                                                              Automation]
       │                                                                  │
       └─renders result inline from query params               [Pipedrive Campaigns]
                                                              (transactional + nurture)
```

The funnel has **one webhook**. Both LM1 and LM2 forms POST the same Make.com endpoint, distinguished by a `magnet` field (`"fthb_lm1"` or `"fthb_lm2"`). The boundary of responsibilities is strict:

- **Astro site (vanilla JS only)** owns: rendering all pages, running the scoring engine client-side, building the result-page URL with plain query params, firing analytics events directly from the browser, and POSTing the form payload to Make.com. There is no backend, no cookies, no `LocalStorage`/`SessionStorage` reliance for cross-page resumption, and no environment variables: the Make.com webhook URL, Pipedrive isn't called directly, and any analytics key are baked into the build.
- **Make.com** owns: receiving the webhook, writing a Google Sheet fallback row, and creating or updating a Pipedrive Person/Lead with the tier, score, magnet, source, and answer fields. Make.com does **not** send email and does **not** route nurture sequences.
- **Pipedrive (with the Campaigns addon)** owns: contact storage, email sending (transactional + nurture), sequence enrollment, sequence pausing/unenrolling, and the email-side conditional logic. All sequence routing is implemented as **Pipedrive Workflow Automations** that trigger off custom fields/labels Make.com sets on the contact (e.g., `fthb_lm1_tier`, `fthb_received_lm2`).

Two non-obvious cross-magnet rules — easy to miss when editing either spec in isolation. Make.com sets the field; Pipedrive Workflow Automations enforce the behavior:

1. When `source = "fthb_lm1_tier_b"` arrives on the LM2 webhook, Make.com sets `fthb_received_lm2 = true` on the existing Pipedrive contact. A Pipedrive automation then **unenrolls** them from the FTHB LM1 Tier B campaign and enrolls them in the FTHB LM2 campaign, so the contact does not get a double email storm.
2. Tier C contacts are **never** pitched the BSS from any sequence; their CTA is education + bi-weekly nurture indefinitely. Encode this as a hard branch in the Pipedrive automation, not as something the email copy has to remember.

The result page (`/orlando-homebuying-readiness-quiz/result`) is **fully static**, driven by **plain query params** (`?n=Maria&t=B&s=68`). No HMAC, no signed token, no env vars (the site has none). The data the page renders is the visitor's own self-reported answers, so tampering with the URL only changes what *they* see on *their* screen; Pipedrive holds the authoritative record from the Make.com webhook. This keeps Make.com out of the render path and lets the page work as a pure static asset.

## Funnel namespace convention (load-bearing)

Every identifier that lives in a globally-shared system — Make.com webhook routing, Pipedrive custom fields, Pipedrive Campaign names, analytics events, the host site's TypeScript module exports — is prefixed with a **funnel slug** so a future second funnel (sellers, investors, relocation, etc.) cannot collide with this one.

**Current funnel slug: `fthb`** (first-time homebuyer). The term already appears throughout the docs and content (`FTHB-range inventory`, `medianPriceFthb`); this just formalizes it as the cross-system namespace.

### What gets prefixed

| Surface | Examples |
|---|---|
| Payload `magnet` discriminator | `"fthb_lm1"`, `"fthb_lm2"` |
| Payload `source` values | `"fthb_lm1_tier_b"`, `"fthb_lm2_standalone"` |
| Pipedrive custom fields on Person | `fthb_lm1_tier`, `fthb_lm1_display_score`, `fthb_lm1_retaken_at`, `fthb_received_lm1`, `fthb_received_lm2`, `fthb_lm2_received_at`, `fthb_lm2_source`, `fthb_q1_credit_range` … `fthb_q10_lender` |
| Pipedrive Campaign names | `FTHB LM1 - Tier A`, `FTHB LM1 - Tier B`, `FTHB LM1 - Tier C`, `FTHB LM2 - Roadmap`, `FTHB Monthly Market Update` |
| Analytics event names | `fthb_readiness_landing_view`, `fthb_readiness_quiz_start`, `fthb_readiness_question_answered`, `fthb_readiness_email_gate_shown`, `fthb_readiness_email_gate_submit`, `fthb_readiness_result_view`, `fthb_readiness_cta_click`, `fthb_roadmap_landing_view`, `fthb_roadmap_optin_view`, `fthb_roadmap_optin_submit`, `fthb_roadmap_view_view`, `fthb_roadmap_step_expand`, `fthb_roadmap_bss_cta_click` |
| URL query param **values** (not names) | `?src=fthb_lm1_tier_b` |
| TypeScript types in the funnel module | `FthbLm1Payload`, `FthbLm2Payload`, `FthbWebhookPayload` |

### What does NOT get prefixed

| Surface | Why |
|---|---|
| Conceptual names in prose: `LM1`, `LM2`, `Tier A/B/C`, `BSS` | These are project-internal terminology; never serialized into another system. |
| Tier enum values: `READY_NOW`, `NINETY_DAY`, `FOUNDATION` | Live inside the prefixed `fthb_lm1_tier` field; the field name is the namespace. |
| Override IDs: `creditUnknownOrLow`, `highDTI`, `exploringTimeline`, `betweenJobs` | Internal to the scoring engine; never crosses a system boundary. |
| Stable answer enum values: `'740_plus'`, `'680_739'`, etc. | Nested inside `answers.qN_*` in the payload; scoped by `magnet`. |
| JSON payload keys nested under `answers.*` or `scoring.*` | Already scoped by the `magnet` discriminator on the same object. The Pipedrive *field name* storing each answer is prefixed (`fthb_q1_credit_range`); the JSON *key* under `answers` is not (`q1_credit_range`). |
| Route paths: `/orlando-homebuying-readiness-quiz`, `/orlando-homebuying-roadmap` | Already specific to FTHB by name; not an identifier in a shared namespace. |
| URL query param **names**: `?n=`, `?t=`, `?s=`, `?src=`, `?preview=` | Scoped to the page reading them; only the *values* of `src` cross into Make.com / Pipedrive. |

### Adding a future funnel

When a second funnel ships (e.g., a sellers funnel), pick a new slug (e.g., `seller`) and apply the same pattern: `seller_lm1`, `seller_lm1_tier`, `seller_received_lm1`, `SELLER LM1 - Tier A`, `seller_readiness_landing_view`, etc. The two funnels then coexist in Pipedrive and Make.com without ambiguity.

## Configuration principle (load-bearing)

Any value flagged "calibrate against real data" in the specs — tier thresholds, per-question point values, override actions, copy strings — **must live in a single config module** (the specs suggest `src/config/readiness.ts` or a JSON variant). Inline-hardcoding any of these will require a refactor in Phase 4 when thresholds get tuned against real submission distributions. See the "Configuration" section of [02-readiness-filter-spec.md](docs/02-readiness-filter-spec.md) for the full list.

Structural constants (e.g., "there are 10 questions") may be inline. The rule of thumb: if the spec has a "this is our current guess" comment next to a value, it belongs in config.

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
- **Agent's edge**: analytical / data-fluent, deep local knowledge. Copy should sound like a diagnostician, not a salesperson.

## Scoring engine specifics (when you build it)

- Raw max is 89; display is normalized to 100 via `round((raw / 89) * 100)`.
- Tier overrides apply **after** the threshold lookup, in the order listed in [02-readiness-filter-spec.md](docs/02-readiness-filter-spec.md). Once a tier has been demoted, do **not** re-promote it.
- The 6 canonical scoring examples the engine must reproduce are in [08-implementation-roadmap.md:97-105](docs/08-implementation-roadmap.md#L97-L105). Each tier and each override has at least one canonical case. Walk through them by hand on real submissions during QA.
- Implement the scorer as a pure TypeScript function that reads config; no UI dependencies.

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

- Backend/data-model code (Make.com + Pipedrive are the backend)
- Cookies, `LocalStorage`/`SessionStorage` for cross-page state, environment variables, or any frontend framework beyond Astro + vanilla JS
- Self-hosted CRM, custom calendar tool, buyer portal, or native mobile apps
- The BSS offer doc itself (next deliverable after this funnel ships)

## Build commands

None yet — there is no Astro project scaffolded. When Phase 1 starts (see [08-implementation-roadmap.md](docs/08-implementation-roadmap.md)), the expected commands will be standard Astro (`npm run dev`, `npm run build`). Tailwind is integrated via the Astro Tailwind integration; no PostCSS config of our own. Update this section once `package.json` exists.
