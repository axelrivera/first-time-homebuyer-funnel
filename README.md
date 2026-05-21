# First-Time Buyer Funnel

Specs, build tasks, and email copy for a two-step lead-generation funnel: a newly licensed Orlando real estate agent's system for moving first-time buyers from initial signal to a 30-minute strategy call.

The strategy is built strictly on Alex Hormozi's frameworks from *$100M Offers* and *$100M Leads*. No paid ads, no cold lists. Every piece of this funnel is a mini Grand Slam Offer delivered through organic warm outreach and free content.

---

## Repository status

**Docs and build tasks only.** No Astro project is scaffolded in this repo. When code ships, it lands in the existing **axelrivera.com** Astro site, not here. This repo holds the specs, the per-task build instructions, and the finalized email copy that gets pasted into Pipedrive.

See [CLAUDE.md](./CLAUDE.md) for the full working context (precedence rules, namespace conventions, hard constraints).

---

## Top-level layout

| Path | What it is |
|---|---|
| [docs/](./docs/) | Strategic source-of-truth specs and content drafts. Numbered for read order. |
| [requirements/](./requirements/) | Sequential, self-contained build tasks. One Claude Code session per file. Active during build; archival after. |
| [email-drafts/](./email-drafts/) | Final shipping copy for every email (LM1 Tier A/B/C + LM2 Roadmap), organized by sequence. This is what gets pasted into Pipedrive Campaigns. |
| [automation-flow.md](./automation-flow.md), [automation-flow-makecom.md](./automation-flow-makecom.md) | Two alternative architecture diagrams (Pipedrive-driven vs. Make.com-driven). Reference only. The canonical model (Make.com orchestrates, Pipedrive runs four linear automations) is captured in CLAUDE.md and the v2 email docs. |
| [CLAUDE.md](./CLAUDE.md) | Agent working instructions: precedence, namespace, hard constraints, advisory modes. |

---

## Funnel flow at a glance

The funnel is a **2-week-to-2-month nurture system** built on a single anchor offer: a 30-minute **strategy call** (the Buyer Strategy Session, or BSS). Every contact enters through one of two on-ramps (the LM1 quiz or the LM2 standalone opt-in), gets segmented by readiness, and runs through a finite email sequence that ends in either the strategy-call pitch or a graceful handoff to a monthly newsletter.

### LM1 quiz → tier assignment

Visitor completes the 10-question Readiness Score and is sorted into one of three tiers. Everyone gets an immediate score-delivery email with their tier guide before tier-specific nurture begins.

### Tier A — Ready Now (14 days, 5 emails)

**Goal: book the strategy call.** Contact already has the basics in place (credit, savings, pre-approval direction). Nurture emails answer the questions that come up between "I'm ready" and "I'm sitting at a closing table": area tradeoffs, what to ask a lender, inspection contingency tactics, a final low-pressure close. Every email points at the strategy call.

### Tier B — 90-Day Sprint (14 days, 5 emails)

**Goal: download the Roadmap.** Contact needs structure before the strategy call makes sense; the Roadmap gives them the full process map. Nurture emails close the gaps that hold 90-day buyers back: order of operations (lender before house-hunting), how to actually rate-shop, HOA/CDD math, the buyer-agent contract conversation. Every email funnels toward the Roadmap. The Roadmap sequence then takes over and pitches the strategy call.

### Tier C — Foundation Phase (8 weeks, 9 emails, weekly)

**Goal: educate, with a graceful exit ramp.** Contact is six or more months out; pitching the strategy call now would burn the relationship. Weekly emails walk through the actual foundation work: credit, savings, down-payment myths, lease-renewal timing, DTI, FHA vs. conventional, Florida down-payment assistance. The final wrap-up email offers the Roadmap as the next read **for contacts who feel they've crossed into 90-Day Sprint territory** during the 8 weeks; everyone else stays in the relationship without a push. **The strategy call is never pitched to Tier C.**

### LM2 Roadmap (14 days, 5 emails)

Entered either from a Tier B graduate or directly from the standalone Roadmap landing page. **Goal: educate on the actual buying process, then pitch the strategy call.** The first email delivers the 9-Step Roadmap; nurture emails answer the questions the Roadmap leaves open (area-specific questions, what to bring to the strategy call, a personal Altamonte story, a final low-pressure note). The strategy-call pitch lands in the last email.

Runs in parallel with whatever tier sequence a contact is already in. Both end on their own; no interaction between them.

### After the funnel ends

Every sequence has a defined end. After the last email, the contact drops out of active nurture and the agent reviews them by hand. Default next step: roll them into a monthly market-update newsletter, the only indefinite channel in the funnel.

### The big picture

| Path | Length | Where it converges |
|---|---|---|
| LM1 → Tier A → strategy call | ~14 days | Strategy call |
| LM1 → Tier B → Roadmap → strategy call | ~28 days (Tier B 14d + Roadmap 14d) | Strategy call |
| LM1 → Tier C → (optional Roadmap → strategy call) | 8 weeks, plus optional 14d Roadmap | Monthly newsletter; strategy call only if Tier C opts into the Roadmap |
| LM2 standalone → strategy call | ~14 days | Strategy call |

One funnel, one anchor offer, four entry paths. Warm contacts close in two weeks. Cold contacts get two months of respect and education before they age out into the monthly newsletter.

---

## How to use the docs

All specs live in [docs/](./docs/). Read in order. Each document has a single responsibility. If something is ambiguous or contradicted across files, the **product specs** (`02-`, `05-`, `09-`, `12-`) win for behavior; the **content drafts** (`03-`, `06-`, `10-`) win for copy. The **email v2 files** (`04-v2`, `07-v2`) own transactional and nurture email; the v1 files are retained for historical reference but were superseded to respect Pipedrive's no-unenroll constraint.

| # | File | What's in it |
|---|---|---|
| 01 | [Strategy & Funnel Overview](./docs/01-strategy-and-funnel.md) | The two-step funnel logic. Why it works. Hormozi value equation applied. How the magnets hand off to each other. |
| 02 | [LM1 Readiness Filter: Product Spec](./docs/02-readiness-filter-spec.md) | UX flow, all 10 scorecard questions, scoring math, tier thresholds, every screen's copy and CTA. |
| 03 | [LM1 Readiness Filter: Content Drafts](./docs/03-readiness-filter-content.md) | The 3 tier results pages, the 2-mistake warnings, the Orlando market snapshot. Copy that ships. |
| 04 | [LM1 Readiness Filter: Email Sequence (v2)](./docs/04-readiness-filter-emails-v2.md) | **Canonical.** Transactional delivery + tier-specific nurture, redesigned around four linear automations. ([v1](./docs/04-readiness-filter-emails.md) retained for reference.) |
| 05 | [LM2 Process Map: Product Spec](./docs/05-process-map-spec.md) | UX flow for the 9-step roadmap opt-in. Entry points. Screen-by-screen. |
| 06 | [LM2 Process Map: Content Drafts](./docs/06-process-map-content.md) | Full content: 9-step roadmap, 3 money-loss mistakes, Orlando gotchas, pre-approval cheat sheet. |
| 07 | [LM2 Process Map: Email Sequence (v2)](./docs/07-process-map-emails-v2.md) | **Canonical.** Roadmap automation (4 emails / 14 days), runs parallel to any Tier automation. ([v1](./docs/07-process-map-emails.md) retained for reference.) |
| 08 | [Implementation Roadmap](./docs/08-implementation-roadmap.md) | Phased build plan. Milestones. What ships first. Definition of done per phase. Canonical scoring examples. |
| 09 | [BSS: Offer Spec](./docs/09-bss-offer-spec.md) | The Buyer Strategy Session as a Grand Slam Offer. Run-of-show, eligibility, Pipedrive routing, hard rules. |
| 09 | [Deal Pipeline Stages](./docs/09-deal-pipeline-stages.md) | The six Pipedrive Deal stages (Quiz Taken → Closing), probability/rotting values, forward-only progression, Make.com stage-setting logic. |
| 10 | [BSS: Content Drafts](./docs/10-bss-content.md) | CTA copy fragments, in-call script (block-by-block), Shortlist PDF template, Lender Comparison Card, Red-Flag Property Filter. |
| 11 | [BSS: Email Sequence](./docs/11-bss-emails.md) | Optional post-call follow-up template. No automated BSS email sequence by design; Google Calendar handles booking/reminder natively. |
| 12 | [BSS: Math and Shortlist](./docs/12-bss-math-and-shortlist.md) | Math sheet structure and formulas, payment / tax / insurance / closing-cost math, shortlist decision logic, prospect-has-own-zip handling. |

---

## Stack reminder (build assumptions baked into specs)

- **Frontend:** Astro + **Tailwind CSS**. Vanilla JavaScript only. Islands where needed for the scorecard interaction. No frontend framework beyond Astro. The funnel adapts to the existing axelrivera.com site's `BaseLayout`, analytics helper, and folder conventions; see [requirements/EXISTING-SITE-NOTES.md](./requirements/EXISTING-SITE-NOTES.md) once it exists (produced by [requirements/00-foundations/01-audit-existing-site.md](./requirements/00-foundations/01-audit-existing-site.md)).
- **Storage on the client:** No cookies, no `localStorage`. `sessionStorage` is used in exactly one narrow place: on LM1 quiz submit the contact's `first_name` and `email` are written so the LM2 Roadmap opt-in form can pre-fill safely if the user continues from a Tier B result page to the Roadmap. `sessionStorage` clears when the tab closes. The quiz state machine itself is in-memory only; reloading mid-quiz restarts.
- **Environment variables:** None. The site is purely static. The Make.com webhook URL and any analytics key are baked into the build. There is no shared secret available for HMAC or anything similar, so the result page is driven by plain query params, not signed tokens.
- **Form handler:** Both LM1 and LM2 forms POST to **one** Make.com webhook URL, distinguished by a `magnet` field (`"fthb_lm1"` or `"fthb_lm2"`). Make.com routes the contact to a Pipedrive Person plus either a Lead (Tier C) or a Deal (Tier A, Tier B, Roadmap), writes all funnel custom fields, and enrolls the contact in the right automation. Make.com also writes a Google Sheet audit row and does **not** send email.
- **CRM + email delivery:** **Pipedrive** (with the **Campaigns** addon). All routing logic lives in Make.com; Pipedrive runs **four linear automations** of single-email campaigns: `FTHB LM1 - Tier A`, `FTHB LM1 - Tier B`, `FTHB LM1 - Tier C`, `FTHB LM2 - Roadmap`. Plus a one-step `Email 0 - LM1 Transactional` automation and the manually-populated `FTHB Monthly Market Update` newsletter. No branching, no mid-flow checks, no programmatic unenrollment.
- **PDF for LM2:** A static PDF hosted at a stable path on the Astro site (e.g., `/assets/orlando-9-step-roadmap.pdf`). The transactional email links to it; no email attachment. Updates are a redeploy.
- **Result page:** Inline rendered immediately on submit, driven by plain query params (`?n=Maria&t=B&s=68`). No DB lookup, no token verification. The user has just submitted their own self-reported answers; the URL is the rendering input. Pipedrive holds the authoritative record.
- **Analytics:** Client-side event tracking via a privacy-friendly provider (Plausible or PostHog). Script tag in the Astro base layout. Events fire directly from the browser, not relayed through Make.com.
- **BSS scheduling:** The agent's existing **Google Calendar appointment scheduling page** (video by default). All BSS CTAs also offer call/text to **(407) 227-3205** as an in-person alternative. No third-party calendar integration.

---

## Naming conventions used throughout

### Conceptual names (in prose, never serialized)

- **LM1** = Lead Magnet 1, the *Readiness Filter*
- **LM2** = Lead Magnet 2, the *Process Map*
- **Tier A / Tier B / Tier C** = "Ready Now" / "90-Day Sprint" / "Foundation Phase" (used interchangeably; specs prefer the names, code can use the tiers)
- **BSS** = Buyer Strategy Session (the 30-minute free consult; the offer at the end of the funnel)

### Funnel namespace (load-bearing; see CLAUDE.md for the full table)

Every identifier that lives in a globally-shared system (Make.com routing, Pipedrive custom fields, Pipedrive automation names, analytics events, TypeScript types) is prefixed with the **funnel slug** `fthb`. This keeps the funnel from colliding with future funnels (sellers, investors, etc.) in the same Pipedrive account and analytics dashboard.

- `magnet` values: `"fthb_lm1"`, `"fthb_lm2"`
- `source` values: `"fthb_lm1_tier_b"`, `"fthb_lm2_standalone"`
- Pipedrive custom fields (mirrored on Lead + Deal, not on Person): `fthb_lm1_tier`, `fthb_received_lm1`, `fthb_received_lm2`, `fthb_q1_credit_range` … `fthb_q10_lender`, etc.
- Pipedrive automations: `FTHB LM1 - Tier A` / `B` / `C`, `FTHB LM2 - Roadmap`, `Email 0 - LM1 Transactional`, `FTHB Monthly Market Update`
- Analytics events: `fthb_readiness_*` (LM1), `fthb_roadmap_*` (LM2)
- TypeScript types: `FthbLm1Payload`, `FthbLm2Payload`, `FthbWebhookPayload`

What does **not** get prefixed: conceptual names (`LM1`, `Tier A`, `BSS`), tier enum values (`READY_NOW`, `NINETY_DAY`, `FOUNDATION`), scoring-engine override IDs, JSON keys nested under `answers.*` or `scoring.*`, and route paths. The full distinction lives in CLAUDE.md under "Funnel namespace convention."

---

## What's NOT in this repo (intentionally out of scope)

- Backend/data-model engineering specs (Make.com + Pipedrive are the backend; we are not building a Rails app for this)
- Cookies, `localStorage`, environment variables, or any frontend framework beyond Astro + vanilla JS (`sessionStorage` is used only for the narrow LM1 → LM2 prefill bridge — see Stack reminder)
- The Astro project itself (it lives in the existing axelrivera.com repo, not here)
- Spanish-language versions of the BSS surfaces (intake form, landing page, in-call script, post-call emails, Shortlist PDF). Deferred to a later iteration per [09-bss-offer-spec.md](./docs/09-bss-offer-spec.md). First launch is English only.
- The active-client onboarding workflow that begins after a prospect signs the Buyer Brokerage Agreement (BBA) (out of scope of the BSS spec)
