# Two-Step Funnel: Source of Truth

This repository is the single source of truth for **Option A: The Two-Step Funnel**, the lead-generation system for a newly licensed Orlando real estate agent targeting first-time buyers.

The strategy is built strictly on Alex Hormozi's frameworks from *$100M Offers* and *$100M Leads*. No paid ads, no cold lists. Every piece of this funnel is a mini Grand Slam Offer delivered through organic warm outreach and free content.

---

## How to use this repo

All specs live in [docs/](./docs/). Read in order. Each document has a single responsibility. If something is ambiguous or contradicted across files, the **product specs** (`02-`, `05-`, `09-`, `12-`) win for behavior; the **content drafts** (`03-`, `06-`, `10-`) win for copy. Email files (`04-`, `07-`, `11-`) own transactional and nurture email content.

| # | File | What's in it |
|---|---|---|
| 01 | [Strategy & Funnel Overview](./docs/01-strategy-and-funnel.md) | The two-step funnel logic. Why it works. Hormozi value equation applied. How the magnets hand off to each other. |
| 02 | [LM1 Readiness Filter: Product Spec](./docs/02-readiness-filter-spec.md) | UX flow, all 10 scorecard questions, scoring math, tier thresholds, every screen's copy and CTA. |
| 03 | [LM1 Readiness Filter: Content Drafts](./docs/03-readiness-filter-content.md) | The 3 tier results pages, the 2-mistake warnings, the Orlando market snapshot. Copy that ships. |
| 04 | [LM1 Readiness Filter: Email Sequence](./docs/04-readiness-filter-emails.md) | Transactional delivery email + tier-specific nurture (5 for Tier A, 6 for Tier B, bi-weekly indefinite for Tier C). |
| 05 | [LM2 Process Map: Product Spec](./docs/05-process-map-spec.md) | UX flow for the 9-step roadmap opt-in. Entry points. Screen-by-screen. |
| 06 | [LM2 Process Map: Content Drafts](./docs/06-process-map-content.md) | Full content: 9-step roadmap, 3 money-loss mistakes, Orlando gotchas, pre-approval cheat sheet. |
| 07 | [LM2 Process Map: Email Sequence](./docs/07-process-map-emails.md) | Transactional delivery + nurture with Buyer Strategy Session CTA. |
| 08 | [Implementation Roadmap](./docs/08-implementation-roadmap.md) | Phased build plan. Milestones. What ships first. Definition of done per phase. |
| 09 | [BSS: Offer Spec](./docs/09-bss-offer-spec.md) | The Buyer Strategy Session as a Grand Slam Offer. Run-of-show, eligibility, Pipedrive routing, hard rules. |
| 10 | [BSS: Content Drafts](./docs/10-bss-content.md) | Landing page, intake form, in-call script (block-by-block), Shortlist PDF template, Lender Comparison Card, Red-Flag Property Filter. |
| 11 | [BSS: Email Sequence](./docs/11-bss-emails.md) | Booking confirmation, T-24h / T-1h reminders, post-call deliverables, day-7 check-in, not-ready-yet redirect, no-show follow-ups. |
| 12 | [BSS: Math and Shortlist](./docs/12-bss-math-and-shortlist.md) | Math sheet structure and formulas, payment / tax / insurance / closing-cost math, shortlist decision logic, prospect-has-own-zip handling. |

---

## Stack reminder (build assumptions baked into specs)

- **Frontend:** Astro + **Tailwind CSS**. Vanilla JavaScript only. Islands where needed for the scorecard interaction. No frontend framework beyond Astro.
- **Storage on the client:** None. No cookies. No `LocalStorage` or `SessionStorage` cross-page resumption. The quiz holds state in memory for one session; if the user reloads, they restart. This is by design.
- **Environment variables:** None. The site is purely static. The Make.com webhook URL and any analytics key are baked into the build. There is no shared secret available for HMAC or anything similar — the result page is driven by plain query params, not signed tokens.
- **Form handler:** Both LM1 and LM2 forms POST to **one** Make.com webhook URL, distinguished by a `magnet` field (`"fthb_lm1"` or `"fthb_lm2"`). Make.com creates/updates a **Pipedrive** Person (with the Campaigns addon installed) and writes a Google Sheet audit row. Make.com does **not** send email.
- **CRM + email delivery:** **Pipedrive** (with the **Campaigns** addon). All transactional and nurture email is sent from Pipedrive Campaigns, triggered by **Pipedrive Workflow Automations** that fire off custom fields Make.com sets (`fthb_lm1_tier`, `fthb_received_lm2`, etc.).
- **PDF for LM2:** A static PDF hosted at a stable path on the Astro site (e.g., `/assets/orlando-9-step-roadmap.pdf`). The transactional email links to it; no email attachment. Updates are a redeploy.
- **Result page:** Inline rendered immediately on submit, driven by plain query params (`?n=Maria&t=B&s=68`). No DB lookup, no token verification. The user has just submitted their own self-reported answers; the URL is the rendering input. Pipedrive holds the authoritative record.
- **Analytics:** Client-side event tracking via a privacy-friendly provider (Plausible or PostHog). Script tag in the Astro base layout. Events fire directly from the browser, not relayed through Make.com.
- **Calendar:** Cal.com or Calendly link for the BSS. No integration; just a URL.

---

## Naming conventions used throughout

### Conceptual names (in prose, never serialized)

- **LM1** = Lead Magnet 1, the *Readiness Filter*
- **LM2** = Lead Magnet 2, the *Process Map*
- **Tier A / Tier B / Tier C** = "Ready Now" / "90-Day Sprint" / "Foundation Phase" (used interchangeably; specs prefer the names, code can use the tiers)
- **BSS** = Buyer Strategy Session (the 30-minute free consult; the offer at the end of the funnel)

### Funnel namespace (load-bearing; see CLAUDE.md for the full table)

Every identifier that lives in a globally-shared system (Make.com routing, Pipedrive custom fields, Pipedrive Campaign names, analytics events, TypeScript types) is prefixed with the **funnel slug** `fthb`. This keeps the funnel from colliding with future funnels (sellers, investors, etc.) in the same Pipedrive account and analytics dashboard.

- `magnet` values: `"fthb_lm1"`, `"fthb_lm2"`
- `source` values: `"fthb_lm1_tier_b"`, `"fthb_lm2_standalone"`
- Pipedrive custom fields: `fthb_lm1_tier`, `fthb_received_lm1`, `fthb_received_lm2`, `fthb_q1_credit_range` … `fthb_q10_lender`, etc.
- Pipedrive Campaigns: `FTHB LM1 - Tier A` / `B` / `C`, `FTHB LM2 - Roadmap`, `FTHB Monthly Market Update`
- Analytics events: `fthb_readiness_*` (LM1), `fthb_roadmap_*` (LM2)
- TypeScript types: `FthbLm1Payload`, `FthbLm2Payload`, `FthbWebhookPayload`

What does **not** get prefixed: conceptual names (`LM1`, `Tier A`, `BSS`), tier enum values (`READY_NOW`, `NINETY_DAY`, `FOUNDATION`), scoring-engine override IDs, JSON keys nested under `answers.*` or `scoring.*`, and route paths. The full distinction lives in CLAUDE.md under "Funnel namespace convention."

---

## What's NOT in this repo (intentionally out of scope)

- Backend/data-model engineering specs (Make.com + Pipedrive are the backend; we are not building a Rails app for this)
- Cookies, `LocalStorage`/`SessionStorage` cross-page state, environment variables, or any frontend framework beyond Astro + vanilla JS
- Spanish-language versions of the BSS surfaces (intake form, landing page, in-call script, post-call emails, Shortlist PDF). Deferred to a later iteration per [09-bss-offer-spec.md](./docs/09-bss-offer-spec.md). First launch is English only.
- The active-client onboarding workflow that begins after a prospect signs the buyer's representation agreement (out of scope of the BSS spec)
