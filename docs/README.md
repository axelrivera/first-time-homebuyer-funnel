# Two-Step Funnel: Source of Truth

This folder is the single source of truth for **Option A: The Two-Step Funnel**, the lead-generation system for a newly licensed Orlando real estate agent targeting first-time buyers.

The strategy is built strictly on Alex Hormozi's frameworks from *$100M Offers* and *$100M Leads*. No paid ads, no cold lists. Every piece of this funnel is a mini Grand Slam Offer delivered through organic warm outreach and free content.

---

## How to use this folder

Read in order. Each document has a single responsibility. If something is ambiguous or contradicted across files, the **product specs** (`02-`, `05-`) win for behavior; the **content drafts** (`03-`, `06-`) win for copy.

| # | File | What's in it |
|---|---|---|
| 01 | [Strategy & Funnel Overview](./01-strategy-and-funnel.md) | The two-step funnel logic. Why it works. Hormozi value equation applied. How the magnets hand off to each other. |
| 02 | [LM1 Readiness Filter: Product Spec](./02-readiness-filter-spec.md) | UX flow, all 10 scorecard questions, scoring math, tier thresholds, every screen's copy and CTA. |
| 03 | [LM1 Readiness Filter: Content Drafts](./03-readiness-filter-content.md) | The 3 tier results pages, the 2-mistake warnings, the Orlando market snapshot. Copy that ships. |
| 04 | [LM1 Readiness Filter: Email Sequence](./04-readiness-filter-emails.md) | Transactional delivery email + tier-specific nurture (5 for Tier A, 6 for Tier B, bi-weekly indefinite for Tier C). |
| 05 | [LM2 Process Map: Product Spec](./05-process-map-spec.md) | UX flow for the 9-step roadmap opt-in. Entry points. Screen-by-screen. |
| 06 | [LM2 Process Map: Content Drafts](./06-process-map-content.md) | Full content: 9-step roadmap, 3 money-loss mistakes, Orlando gotchas, pre-approval cheat sheet. |
| 07 | [LM2 Process Map: Email Sequence](./07-process-map-emails.md) | Transactional delivery + nurture with Buyer Strategy Session CTA. |
| 08 | [Implementation Roadmap](./08-implementation-roadmap.md) | Phased build plan. Milestones. What ships first. Definition of done per phase. |

---

## Stack reminder (build assumptions baked into specs)

- **Frontend:** Astro (static + islands where needed for the scorecard interaction)
- **Form handler:** Astro form posts to a **Make.com webhook**
- **Email delivery:** Make.com scenario → email provider (decision deferred; see `08-implementation-roadmap.md`)
- **PDF generation:** Generated server-side in Make.com OR pre-built static PDFs (TBD per asset)
- **Result page:** Inline rendered immediately on submit + email copy follow-up
- **Analytics:** Event-level tracking on submit, tier assignment, CTA clicks (provider TBD)

---

## Naming conventions used throughout

- **LM1** = Lead Magnet 1, the *Readiness Filter*
- **LM2** = Lead Magnet 2, the *Process Map*
- **Tier A / Tier B / Tier C** = "Ready Now" / "90-Day Sprint" / "Foundation Phase" (used interchangeably; specs prefer the names, code can use the tiers)
- **BSS** = Buyer Strategy Session (the 30-minute free consult; the offer at the end of the funnel)

---

## What's NOT in this folder (intentionally out of scope)

- Backend/data-model engineering specs (using Make.com, so we are not building a Rails app for this)
- Deep technical integration docs for the email provider (deferred until provider is chosen)
- The Buyer Strategy Session offer doc itself (the next deliverable after this funnel ships)
- Spanish translations of all copy (flagged in the roadmap as Phase 3, not drafted here)
