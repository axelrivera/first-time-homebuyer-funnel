# Two-Step Funnel: Source of Truth

This repository is the single source of truth for **Option A: The Two-Step Funnel**, the lead-generation system for a newly licensed Orlando real estate agent targeting first-time buyers.

The strategy is built strictly on Alex Hormozi's frameworks from *$100M Offers* and *$100M Leads*. No paid ads, no cold lists. Every piece of this funnel is a mini Grand Slam Offer delivered through organic warm outreach and free content.

---

## How to use this repo

All specs live in [docs/](./docs/). Read in order. Each document has a single responsibility. If something is ambiguous or contradicted across files, the **product specs** (`02-`, `05-`) win for behavior; the **content drafts** (`03-`, `06-`) win for copy.

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

---

## Stack reminder (build assumptions baked into specs)

- **Frontend:** Astro + **Tailwind CSS**. Vanilla JavaScript only. Islands where needed for the scorecard interaction. No frontend framework beyond Astro.
- **Storage on the client:** None. No cookies. No `LocalStorage` or `SessionStorage` cross-page resumption. The quiz holds state in memory for one session; if the user reloads, they restart. This is by design.
- **Environment variables:** None. The site is purely static. The Make.com webhook URL and any analytics key are baked into the build. There is no shared secret available for HMAC or anything similar — the result page is driven by plain query params, not signed tokens.
- **Form handler:** Both LM1 and LM2 forms POST to **one** Make.com webhook URL, distinguished by a `magnet` field. Make.com creates/updates a **Pipedrive** Person (with the Campaigns addon installed) and writes a Google Sheet audit row. Make.com does **not** send email.
- **CRM + email delivery:** **Pipedrive** (with the **Campaigns** addon). All transactional and nurture email is sent from Pipedrive Campaigns, triggered by **Pipedrive Workflow Automations** that fire off custom fields Make.com sets (`lm1_tier`, `received_lm2`, `preferred_language`, etc.).
- **PDF for LM2:** A static PDF hosted at a stable path on the Astro site (e.g., `/assets/orlando-9-step-roadmap.pdf`). The transactional email links to it; no email attachment. Updates are a redeploy.
- **Result page:** Inline rendered immediately on submit, driven by plain query params (`?n=Maria&t=B&s=68`). No DB lookup, no token verification. The user has just submitted their own self-reported answers; the URL is the rendering input. Pipedrive holds the authoritative record.
- **Analytics:** Client-side event tracking via a privacy-friendly provider (Plausible or PostHog). Script tag in the Astro base layout. Events fire directly from the browser, not relayed through Make.com.
- **Calendar:** Cal.com or Calendly link for the BSS. No integration; just a URL.

---

## Naming conventions used throughout

- **LM1** = Lead Magnet 1, the *Readiness Filter*
- **LM2** = Lead Magnet 2, the *Process Map*
- **Tier A / Tier B / Tier C** = "Ready Now" / "90-Day Sprint" / "Foundation Phase" (used interchangeably; specs prefer the names, code can use the tiers)
- **BSS** = Buyer Strategy Session (the 30-minute free consult; the offer at the end of the funnel)

---

## What's NOT in this repo (intentionally out of scope)

- Backend/data-model engineering specs (Make.com + Pipedrive are the backend; we are not building a Rails app for this)
- Cookies, `LocalStorage`/`SessionStorage` cross-page state, environment variables, or any frontend framework beyond Astro + vanilla JS
- The Buyer Strategy Session offer doc itself (the next deliverable after this funnel ships)
- Spanish translations of all copy (flagged in the roadmap as Phase 3, not drafted here)
