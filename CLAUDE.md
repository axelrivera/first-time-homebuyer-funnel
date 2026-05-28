# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository status

**Docs-only right now.** No code, no build tooling. The only contents are specs in [docs/](docs/) that define a two-step lead-generation funnel for a newly licensed Orlando real estate agent targeting first-time buyers. When code lands, it will be an **Astro + Tailwind CSS** static site (vanilla JavaScript, no backend, no cookies, no environment variables) that POSTs to a **Make.com** webhook. Make.com creates a Person in **Pipedrive** and enrolls them in the appropriate Pipedrive automation; transactional and nurture email are sent from four linear Pipedrive automations orchestrating single-email campaigns. That stack assumption is baked into every spec.

## What this funnel is

Two lead magnets, gated entry, segmented by readiness. Both terminate at the same offer: a 30-minute **Buyer Strategy Session (BSS)**.

- **LM1** — *The Orlando First-Time Buyer Readiness Score* — 10-question scorecard, scored, segments into 3 tiers.
- **LM2** — *The 10-Step First Home Roadmap* — long-form process map, only promoted to Tier B from LM1 and from one standalone landing page.

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
- **Email files** ([04-readiness-filter-emails-v2.md](docs/04-readiness-filter-emails-v2.md), [07-process-map-emails-v2.md](docs/07-process-map-emails-v2.md)) own transactional + nurture sequences. The v1 files ([04-readiness-filter-emails.md](docs/04-readiness-filter-emails.md), [07-process-map-emails.md](docs/07-process-map-emails.md)) are retained for reference but were superseded by v2 to respect Pipedrive Campaigns' no-unenroll constraint — **always read v2 for current behavior**.
- [08-implementation-roadmap.md](docs/08-implementation-roadmap.md) is the phased build plan and the source of canonical scoring examples.
- [09-deal-pipeline-stages.md](docs/09-deal-pipeline-stages.md) defines the Pipedrive Deal pipeline stages (Quiz Taken, Received Roadmap, BSS Booked, Pre-Approved & Searching, Under Contract, Closing), their probability and rotting values, the forward-only stage-progression rule, and the Make.com stage-setting logic.

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
       └─renders result inline from query params               [4 Pipedrive automations]
                                                              (Tier A/B/C + Roadmap, linear)
```

The funnel has **one webhook**. Both LM1 and LM2 forms POST the same Make.com endpoint, distinguished by a `magnet` field (`"fthb_lm1"` or `"fthb_lm2"`). The boundary of responsibilities is strict:

- **Astro site (vanilla JS only)** owns: rendering all pages, running the scoring engine client-side, building the result-page URL with plain query params, firing analytics events directly from the browser, and POSTing the form payload to Make.com. There is no backend, no cookies, no `localStorage`, and no environment variables: the Make.com webhook URL is baked into the build; Pipedrive isn't called directly. **`sessionStorage` is used in exactly one place**: on LM1 quiz submit, the contact's `first_name` and `email` are written to `sessionStorage` so the LM2 Roadmap opt-in form can pre-fill safely if the user continues from a Tier B result page to the Roadmap. `sessionStorage` clears when the tab closes; the LM1 quiz state machine itself remains in-memory only, so reloading mid-quiz still restarts.
- **Make.com** owns: receiving the webhook, writing a Google Sheet fallback row, creating or updating the right Pipedrive entity (Person + Lead or Deal — see below), writing all funnel fields on that Lead or Deal, **and enrolling the contact in the right Pipedrive automation**. All routing logic in this funnel lives in Make.com; Pipedrive automations are pure linear sequences. Make.com does **not** send email itself.
- **Pipedrive (with the Campaigns addon)** owns: contact storage and email sending. In Pipedrive terminology, a "campaign" is a single email template; an "automation" orchestrates a sequence of those campaigns. Iteration 1 of this funnel uses exactly **four linear automations**:
  - `FTHB LM1 - Tier A` (5 emails / 14 days) — primary entity: **Deal**
  - `FTHB LM1 - Tier B` (3 emails / 5 days) — primary entity: **Deal**
  - `FTHB LM1 - Tier C` (9 emails / 8 weeks) — primary entity: **Lead**
  - `FTHB LM2 - Roadmap` (4 emails / 14 days) — primary entity: **Deal**
  Plus a one-step `Email 0 - LM1 Transactional` automation (fires on whichever entity Make.com just created or updated) and the manually-populated `FTHB Monthly Market Update` newsletter. **None of the automations branch on field state** — they are linear, beginning to end. Routing decisions (which automation to enroll a contact in) live in Make.com.

### Person vs. Lead vs. Deal split (load-bearing)

- **Person** record carries only identity fields: `name`, `email`, `phone`, `marketing_status` (CAN-SPAM consent). Nothing funnel-specific lives on the Person.
- **All FTHB custom fields** (`fthb_lm1_tier`, `fthb_lm1_display_score`, `fthb_received_lm1`, `fthb_received_lm2`, `fthb_lm2_received_at`, `fthb_lm2_source`, `fthb_q1_credit_range` … `fthb_q10_lender`, etc.) are configured as custom fields on **both Lead and Deal** entity types — same field name, same type on both — so Make.com can copy them cleanly when a Lead is promoted to a Deal.
- **Tier C contacts are Leads.** Tier A, Tier B, and Roadmap contacts are Deals. The entity type tracks contact "warmth": cold/educational = Lead, in-pipeline = Deal.
- **Make.com routing decides Lead vs. Deal per webhook:**
  - LM1 webhook with Tier A or Tier B → create/update Deal
  - LM1 webhook with Tier C → create/update Lead
  - LM2 webhook (any source) → create/update Deal
- **Conflict-resolution rules** (Make.com applies before enrollment):
  - Existing Deal + new Deal-worthy event → update the existing Deal (no duplicate)
  - Existing Lead + new Deal-worthy event (e.g., Tier C → Tier B retake, or a Tier C Lead downloads LM2) → **convert the Lead to a Deal**, copy all custom fields, then update
  - Existing Deal + new Lead-worthy event (e.g., Tier B → Tier C retake) → update tier field on the Deal but **stay a Deal** (never demote)
  - No existing record → create per the entity rule above

### Hard tooling constraints (load-bearing — designs that violate these will not run)

- **No branching inside Pipedrive automations.** Pipedrive automations can technically branch, but iteration 1 deliberately doesn't. Every automation is a linear sequence of single-email campaigns. If routing logic is needed (which contact gets which automation, when to suppress an enrollment), it lives in Make.com config — not in a Pipedrive automation step. Do not propose mid-flow checks on `fthb_lm1_tier` or `fthb_received_lm2` inside Pipedrive.
- **No programmatic unenrollment.** Once a contact is in a Pipedrive automation, it can only finish on its own schedule or be stopped manually by the agent from the contact record. Make.com / Pipedrive cannot unenroll a contact programmatically. Every automation must be safe to run to completion. Mid-flight switching does not exist as an option.
- **BSS bookings are invisible to the funnel.** Buyer Strategy Sessions are booked in Google Calendar with no return path to Pipedrive. Any `bss_booked` field is manual-only; do not use it as automation input, enrollment gate, or branch condition. Tier A email content must therefore be safe to finish whether the contact has booked, attended, or never booked — no "did you forget to book?" emails.

Two non-obvious cross-magnet rules — easy to miss when editing either spec in isolation:

1. **Roadmap and Tier B run in parallel without coordination.** The `FTHB LM2 - Roadmap` automation enrolls any contact whose LM2 webhook fires, regardless of whether they're already in `FTHB LM1 - Tier B`. Both automations are short and finite (Tier B is 3 emails / 5 days; Roadmap is 4 emails / 14 days), so the overlap is bounded and acceptable. The agent can add a Make.com suppression to skip Roadmap enrollment for already-in-Tier-B contacts as a future tightening, but iteration 1 doesn't bother. See [docs/04-readiness-filter-emails-v2.md](docs/04-readiness-filter-emails-v2.md) and [docs/07-process-map-emails-v2.md](docs/07-process-map-emails-v2.md).
2. Tier C contacts are **never** pitched the BSS from any sequence — Tier C automation doesn't pitch it, Tier C welcome doesn't pitch it. The only edge where a Tier C contact could see a BSS pitch is if Make.com enrolls them in the Roadmap automation (because they downloaded LM2 via the standalone landing page after taking the quiz). The mitigation is at the Make.com layer: configure the LM2 scenario to skip Roadmap enrollment when `fthb_lm1_tier = FOUNDATION`, if/when this case actually shows up in practice.

The result page (`/orlando-homebuying-readiness-quiz/result`) is **fully static**, driven by **plain query params** (`?n=Maria&t=B&s=68`). No HMAC, no signed token, no env vars (the site has none). The data the page renders is the visitor's own self-reported answers, so tampering with the URL only changes what *they* see on *their* screen; Pipedrive holds the authoritative record from the Make.com webhook. This keeps Make.com out of the render path and lets the page work as a pure static asset.

## Funnel namespace convention (load-bearing)

Every identifier that lives in a globally-shared system — Make.com webhook routing, Pipedrive custom fields, Pipedrive automation names, analytics events, the host site's TypeScript module exports — is prefixed with a **funnel slug** so a future second funnel (sellers, investors, relocation, etc.) cannot collide with this one.

**Current funnel slug: `fthb`** (first-time homebuyer). The term already appears throughout the docs and content (`FTHB-range inventory`, `medianPriceFthb`); this just formalizes it as the cross-system namespace.

### What gets prefixed

| Surface | Examples |
|---|---|
| Payload `magnet` discriminator | `"fthb_lm1"`, `"fthb_lm2"` |
| Payload `source` values | `"fthb_lm1_tier_b"`, `"fthb_lm2_standalone"` |
| Pipedrive custom fields on Lead + Deal (mirror schemas; **not** on Person) | `fthb_lm1_tier`, `fthb_lm1_display_score`, `fthb_lm1_retaken_at`, `fthb_received_lm1`, `fthb_received_lm2`, `fthb_lm2_received_at`, `fthb_lm2_source`, `fthb_q1_credit_range` … `fthb_q10_lender` |
| Pipedrive automation names | `FTHB LM1 - Tier A`, `FTHB LM1 - Tier B`, `FTHB LM1 - Tier C`, `FTHB LM2 - Roadmap`, `Email 0 - LM1 Transactional` (one-step). All linear, no branching. Plus the manually-populated `FTHB Monthly Market Update` newsletter. |
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
- **"The 10-Step First Home Roadmap"** + subhead *"Exactly What Happens Between 'I Think I'm Ready' and Keys in Your Hand in Orlando."*

Also locked: the **stable enum keys** for each scorecard answer (e.g., `q1_credit_range: "680_739"`). Make.com routes on these — renaming them breaks the email scenario. Human-readable labels in the UI can be re-copy-edited freely; the underlying enum keys cannot.

## Marketing link placeholders

Any clickable CTA in copy (email body, landing page button, DM, ad, script) that points to a URL the agent has to wire up — a calendar link, a quiz page, a roadmap page, a PDF download, a retake link — must be written as a **plain-English call-to-action in square brackets** on its own line: `[Book the Buyer Strategy Session]`, `[Read the Roadmap]`, `[Take the quiz]`, `[Download the PDF]`. The bracket text serves three jobs at once: it's the visible link label, the placeholder marker (so the agent knows to swap in the real URL before sending), and the CTA copy itself.

Do **not** use Pipedrive merge-tag syntax (`{{book_bss_link}}`, `{{fthb_roadmap_pdf_link}}`, etc.) in draft copy. Those tags only resolve if the matching merge field is configured in Pipedrive; if it isn't, the literal `{{...}}` ships in the email. Bracket placeholders fail visibly during review instead of silently in the inbox.

Two structural rules:

1. **The bracket placeholder takes its own line.** Never inline a `[CTA]` at the end of a sentence — it reads as footnote text, not a button. (This replaces the older "merge tags take a full line" guidance.)
2. **Don't restate the CTA in the lead-in line.** If the bracket says `[Book the Buyer Strategy Session]`, don't precede it with "Book your Buyer Strategy Session here:" or "Book the session:" — the bracket already names the action. Lead-ins that *add* context are fine and often useful: naming the channel ("Book a video call:") so the reader knows the bracket is the video route, or framing the moment ("If we don't already have a call on the calendar:"). The test: does the lead-in carry information the bracket can't?

Personalization tokens (`*|FIRST_NAME|*`, `{{agent_first_name}}`, etc.) are not marketing links and stay in their native template syntax — the rule above is for *clickable URLs only*.

## Working voice and the Grand Slam Offer test

All user-facing copy — landing pages, quiz screens, result pages, transactional and nurture emails, CTAs — is written in the agent's voice: **direct, warm, educational, never salesy**. Two Hormozi tests every piece of copy must pass before it ships:

- *"Would the prospect feel stupid to say no?"* — the offer must stack enough value that declining feels irrational.
- *"Does this feel like they're getting a deal, not being sold to?"* — lead with usefulness, never with a pitch.

Every offer in the funnel — including LM1, LM2, and every CTA inside them — is a **Grand Slam Offer** evaluated against Hormozi's value equation:

```
Value = (Dream Outcome × Perceived Likelihood of Achievement) / (Time Delay × Effort & Sacrifice)
```

[01-strategy-and-funnel.md](docs/01-strategy-and-funnel.md) already shows this applied to LM1 and LM2. When editing headlines, subject lines, or CTA copy, optimize for **all four** drivers — don't silently trade one for another. Lead magnets are mini-offers: never propose a vague freebie ("Free Buyer's Guide"); always name the specific result and the time-to-result (the LM1 / LM2 locked titles are the template).

## Regulatory compliance (load-bearing — overrides voice, Hormozi, and convenience)

Every piece of copy generated for this funnel — landing pages, quiz screens, result pages, transactional and nurture emails, CTAs, ads, DMs, scripts, and any advisory-mode output — must be compliant with U.S. and Florida real estate law and industry best practices. When a Hormozi-shaped framing collides with a compliance rule, **compliance wins**; rephrase the offer, don't soften the rule. The agent is a licensed Florida REALTOR; copy that goes out under their name is subject to the same constraints whether it ships on a landing page or in a personal text.

Specific rules that bind every copy decision:

- **Fair Housing Act + Florida Fair Housing Act**: No language, imagery, or filtering that steers, prefers, limits, or discourages by protected class — race, color, religion, sex (including sexual orientation and gender identity per HUD 2021 guidance), national origin, familial status, disability, age, or marital status. This rules out neighborhood descriptions framed around demographics ("family-friendly neighborhood," "good schools" as a demographic proxy, "safe area," "up-and-coming"), persona copy that targets or excludes protected classes, and any quiz logic that routes by anything other than the buyer's own stated financial readiness. Persona references in the funnel (e.g., "families relocating from Puerto Rico") describe *who is researching the topic*, never who *should* live somewhere; do not let that internal framing leak into outward copy that suggests certain buyers belong in certain areas.
- **RESPA (Real Estate Settlement Procedures Act)**: No kickbacks, referral fees, or fee-splits with settlement-service providers (lenders, title companies, inspectors, insurance agents, attorneys, appraisers). Copy may **name** specific providers as recommendations when there's a genuine basis; it may **not** imply or disclose any compensation arrangement, exclusive-referral relationship, or "preferred lender" deal. "Affiliated business arrangements" (ABAs) require written disclosure at the moment of referral, which this funnel does not provide — so do not write copy that implies one exists. Lender content stays diagnostic ("here's how to evaluate a lender") not transactional ("use my guy").
- **TILA / Reg Z + MAP Rule (mortgage advertising)**: Any quoted interest rate, APR, payment amount, down-payment percentage, or loan-term claim in copy must be either (a) clearly framed as an illustrative example with disclosed assumptions or (b) sourced and dated. Do not invent rate numbers, payment numbers, or program terms. Do not promise loan approval, specific rates, or specific monthly payments. "You could qualify for..." language without disclosed assumptions is a MAP Rule risk.
- **NAR Code of Ethics + Florida real-estate-license law (Chapter 475, F.S. and FREC rules)**: Brokerage and license identification on emails is handled by the Pipedrive email template chrome (footer, brokerage name, license number) — copy doesn't need to repeat it. What copy *does* control: no statements that imply the agent is the broker, and no false or misleading claims about services, market conditions, or the agent's qualifications. The existing memory rule on the agent's track record (newly licensed; do not gate copy on deal count or years) is partly a 475.25(1)(b) misleading-advertising guardrail, not just a Hormozi preference. For surfaces *outside* email (landing pages, ads, DMs, scripts), the brokerage identification requirement is back in play and copy must carry it.
- **CAN-SPAM + Florida Electronic Mail Communications Act**: Every commercial email must have a valid physical mailing address, a working one-click unsubscribe, no deceptive subject lines, and no scraped/purchased lists. The funnel's email infrastructure (Pipedrive Campaigns) handles unsubscribe and address footer mechanically; copy must not write subject lines or body language that misrepresents the email's nature ("Re:" replies to threads that don't exist, fake forwarded headers, false urgency about non-existent transactions).
- **Florida-specific disclosures**: When discussing the buying process, do not misrepresent statutory rights or invent them. Specifically: the Florida purchase-contract inspection period is what the contract says it is (negotiated, typically 7–15 days); HOA/condo three-day cancellation rights are real but narrowly scoped; the buyer/agent agreement in Florida is the **Buyer Brokerage Agreement (BBA)**, not "Buyer Representation Agreement." If copy describes a Florida statute or contract clause, the description must be accurate to the current Florida Realtors / Florida Bar forms and Florida statutes. When in doubt, name the mechanism in general terms rather than quote a number or clause that might be wrong.
- **No legal, tax, or specific financial advice**: Copy may explain *what* a contingency, contract clause, tax credit, or loan program *is*; it must not tell a specific buyer what to do with their specific financial, legal, or tax situation. Hand off to "talk to your CPA / attorney / loan officer" when copy edges toward individualized advice.
- **Verify before you publish**: Any rate, price, statute, program name, deadline, or regulatory claim in copy must be checked against current sources before the copy ships. Memory rule on validating numbers applies here and is partly a compliance guardrail. "Plausible-sounding" is not a compliance standard.

When copy is generated by Claude in this project, treat these rules as a hard pre-flight checklist — not aspirational. If a draft can't be made compliant without losing the offer, flag the conflict to the agent rather than shipping the offer-shaped version.

## Market context to bake into examples and copy

Default these unless the user tells you otherwise — generic "Florida" examples are a smell.

- **Geography**: Orlando metro, anchored in the **northern half** (Seminole County and north Orange County) with two deliberate outliers — **Winter Garden** on the west and **Lake Nona** in the southeast — to show buyers the full picture of what a $500K budget unlocks. The canonical 10-area set used by the market snapshot and rotated through monthly content is, in price-band order from well-below-anchor to stretch: **Altamonte Springs, Apopka, Sanford, Winter Springs, Longwood, Lake Mary, Oviedo, Maitland, Winter Garden, Lake Nona**. Snapshots, gotchas, and price examples should name areas from this set specifically — do **not** narrow to Seminole-only, default back to the old I-4-corridor framing (Eatonville, Casselberry, Winter Park standalone), reintroduce **College Park or Audubon Park** (both were considered and dropped — medians sit ~$75–100K above the $500K anchor with too-thin entry-tier inventory), or widen to areas the user hasn't named (Belle Isle, Conway, Dr. Phillips, Kissimmee, Clermont, Horizon West, St. Cloud, etc.). Casselberry / Fern Park may still appear as a tactical "border-zone" mention in process-map content, but is not part of the market report rotation.
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
- Cookies, `localStorage`, environment variables, or any frontend framework beyond Astro + vanilla JS. (`sessionStorage` is used only for the narrow LM1 → LM2 prefill bridge described in the architecture section.)
- Self-hosted CRM, custom calendar tool, buyer portal, or native mobile apps
- The BSS offer doc itself (next deliverable after this funnel ships)

## Build commands

None yet — there is no Astro project scaffolded. When Phase 1 starts (see [08-implementation-roadmap.md](docs/08-implementation-roadmap.md)), the expected commands will be standard Astro (`npm run dev`, `npm run build`). Tailwind is integrated via the Astro Tailwind integration; no PostCSS config of our own. Update this section once `package.json` exists.
