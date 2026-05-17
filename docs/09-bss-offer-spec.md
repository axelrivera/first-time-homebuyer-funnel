
This is the offer spec for the **Buyer Strategy Session (BSS)** — the 30-minute conversation that terminates both LM1 (Tier A) and LM2. Everything a developer, marketer, or the agent themselves needs to ship the BSS lives here: the Grand Slam Offer, the 30-minute run-of-show, pre/post-call ops, the Pipedrive namespace, analytics events, and the Workflow Automations that route prospects in and out.

Status: **DRAFT**. Calibration values (slot length, deliverable turnaround windows, no-show retry counts) are flagged in line and belong in `src/config/bss.ts` once the build starts.

---

## v0 MVP cut — what ships first

The spec below describes the steady-state BSS once it has been calibrated against real bookings. **v0** is the manual, low-automation version that ships first. The agent runs the first ~20 BSSes by hand and only promotes pieces of the spec into automation once volume justifies it.

The split is operational, not experiential. From the prospect's side, the v0 call looks identical to the v1 call: same landing page, same guarantee, same 30-minute run-of-show, same deliverables.

### Ships in v0

- **Landing page**, hero through FAQ (see [10-bss-content.md](10-bss-content.md))
- **The 30-Minute Promise** guarantee, verbatim
- **The 30-minute run-of-show**, all 5 blocks, in-call script intact
- **The two flagship bonuses**: the live Google Sheet math built on the call, and the personalized Shortlist PDF sent within 24 hours
- **The Option 1 / 2 / 3 close**
- **Intake form**, trimmed to **4 questions**: partner Y/N, timeline, cash bucket, neighborhoods considering. Hosted on Google Forms or Tally; the agent reads responses in the form's spreadsheet pre-call. No Make.com integration.
- **Calendar**: Cal.com booking only. No Cal.com to Make.com webhook. Cal.com sends its native confirmation and reminder emails.
- **3 emails total**: Cal.com's native booking confirmation, Cal.com's native 1-hour reminder, and **BSS-4 (post-call deliverable)** sent manually by the agent with the Shortlist PDF link attached.
- **3 Pipedrive fields**: `fthb_bss_booked_at`, `fthb_bss_attended`, `fthb_bss_outcome`. Set manually after each call.
- **3 outcome values**: `signed_buyer_agreement`, `shortlist_only`, `no_show`.
- **2 analytics events**: `fthb_bss_landing_view`, `fthb_bss_cta_click`. Bookings and attendance counted by hand from Cal.com plus Pipedrive.
- **Hard rules** (Tier C exclusion, 30-min cap, no paid ads). Tier C exclusion in v0 is enforced by simply not placing a BSS CTA on Tier C result pages.

### Deferred to v1 (after first ~20 calls)

- The **Red-Flag Property Filter PDF**. In v0, the agent mentions the 7 items verbally in Block 4. Produce the PDF once the agent has heard which items prospects react to most.
- The **3-Lender Comparison Card PDF**. In v0, the agent sends a plaintext email with three names, numbers, and one-line "best for" tags. Format the card once the lender bench stabilizes.
- The **Make.com webhook payloads** (both `fthb_bss_intake` and `fthb_bss_booking`). Manual Pipedrive entry until volume justifies the integration.
- All **6 Pipedrive Workflow Automations** (BSS Booking Confirmation, Pause Nurture During BSS Window, Post-call Routing, Tier C Guardrail, No-Show Handling, Reschedule Cap). Hand-route the first 20 contacts.
- The other **5 emails** (BSS-2, BSS-5, BSS-6, BSS-NS-1, BSS-NS-2). Add as the data tells you they're needed (a no-show rate above 15% justifies building BSS-NS-1; a stalled `shortlist_only` cohort justifies BSS-5).
- The other **5 Pipedrive fields** (`fthb_bss_scheduled_for`, `fthb_bss_source`, `fthb_bss_intake_completed_at`, `fthb_bss_followup_sent_at`, `fthb_bss_rescheduled_count`). Add once an automation needs to read them.
- The other **2 outcome values** (`lender_intro_made`, `not_ready_yet`). In v0 they collapse into `shortlist_only`; the agent's call notes capture the nuance.
- The other **5 analytics events** (`fthb_bss_calendar_view`, `fthb_bss_booked`, `fthb_bss_intake_submit`, `fthb_bss_attended`, `fthb_bss_outcome_recorded`). Hand-count from Cal.com plus Pipedrive.
- The `src/config/bss.ts` module. v0 has no code; calibration values live in the math sheet's cell defaults and in this doc.

### Promote-to-v1 trigger

Promote any deferred piece when volume forces it. The rough bar: **5+ booked BSSes/week for 4 weeks running**. Below that, manual is faster than building integrations, and the data needed to calibrate the automation does not exist yet.

The full spec below describes the v1 target state. Treat the sections after this one as the design v0 is iterating toward, not the design v0 ships.

---

## What the BSS is

A free 30-minute one-on-one conversation between the agent and a prospect who has earned the right to be on it. "Earned the right" is a hard rule, not a tone — see "Eligibility" below.

The BSS is **not** a sales call. Per the project rule from [01-strategy-and-funnel.md](01-strategy-and-funnel.md): *"making the customer feel like they're getting a deal, not being sold to."* The call's measurable output is a written **buy-box** and a **3-neighborhood shortlist** the prospect can use with or without this agent. That deliverable is what's being given. The agent's offer to walk them through the actual transaction is what's being asked.

Channel: video call (Cal.com / Zoom link in the calendar invite). Phone is fine if the prospect requests it; in person at a coffee shop is fine if the prospect requests it. The agent does not push for in-person.

Language: English only for the first iteration. Spanish-language delivery is a known future expansion (Puerto Rico relocation buyers are a primary persona per [CLAUDE.md](../CLAUDE.md)) and is deferred until the English funnel proves out — see "What's intentionally out of scope" below.

---

## Public name (proposed)

**The Orlando First-Time Buyer Strategy Session**
*Subhead:* 30 Minutes to Your Personal Buy-Box, a Shortlist of I-4 Corridor Neighborhoods That Fit Your Numbers, and the Exact Next Step — Whether You Hire Me or Not.

This wording is **proposed, not locked**. Treat it as a starting point and run it through the Hormozi voice test ("would the prospect feel stupid to say no?" + "do they feel like they're getting a deal, not being sold to?") before locking. Once locked here, it must match landing pages, calendar tool descriptions, email subject lines, and DM scripts verbatim.

The two locked LM1 / LM2 titles in [CLAUDE.md](../CLAUDE.md) are the template: a specific result + a time-to-result. This name follows that template (specific result = buy-box + shortlist + next step; time-to-result = 30 minutes).

---

## The Grand Slam Offer

### Hormozi value equation applied

> Value = (Dream Outcome × Perceived Likelihood of Achievement) / (Time Delay × Effort & Sacrifice)

| Driver | How the BSS maxes it out |
|---|---|
| **Dream Outcome** | A Tier A first-time buyer wants one thing: certainty about *which neighborhood, at which price, with which lender, on which timeline*. Not "house tours." The BSS gives them a written buy-box and a 3-neighborhood I-4 corridor shortlist sized to their down payment + credit + household — the artifact a confused first-time buyer cannot produce on their own without 40 hours of Zillow + r/RealEstate. |
| **Perceived Likelihood** | The agent is data-fluent, lives on the I-4 corridor, and shows up to the call with the prospect's LM1 scorecard answers already read. The call opens with the agent restating the prospect's own answers back to them. That single move — visible preparation — does more for perceived likelihood than any testimonial. |
| **Time Delay** | 30 minutes to a buy-box and shortlist. Personalized PDF in the prospect's inbox within 24 hours. No "we'll get back to you next week." |
| **Effort & Sacrifice** | 3-minute intake form, sent on booking. No documents to upload. No credit pull. No commitment to use this agent. The prospect can take the buy-box to any agent or any lender afterward and lose nothing. |

### Bonuses (value stack)

Each bonus is something that costs the agent little to produce but is worth a lot to the prospect. None of them are "PDFs already written for everyone" — they are produced *from this prospect's actual answers*.

1. **The 3-Neighborhood Shortlist (PDF, personalized).** Three named ZIP codes between Sanford and Downtown Orlando, selected against the prospect's stated price ceiling, commute, household, and deal-breakers. If the prospect walked in with one or two neighborhoods already in mind, those lead the list and the math runs on them first; the agent's pick fills the remaining slot as an alternate, never as a replacement. Each entry includes the price point, the trade-off, and one current MLS comp under contract. Delivered within 24 hours of the call. Branching logic for the no-preference, has-preference, and has-preference-but-math-does-not-work cases is in [12-bss-math-and-shortlist.md](12-bss-math-and-shortlist.md).
2. **The Live Payment & Closing-Cost Math (Google Sheet, shared link).** Built *on the call* in front of the prospect: their down payment, today's rate, FHA vs. conventional, property tax + insurance + (if applicable) HOA. The prospect leaves the call with the link and can re-run the numbers themselves. This bonus is the diagnostic moment — most first-time buyers have never seen the math, and seeing it on a real address removes the largest source of uncertainty.
3. **The Red-Flag Property Filter (1-page PDF).** Seven deal-breakers specific to the Sanford↔Downtown corridor: failed-permit additions, septic-to-sewer transition zones, roof age vs. insurance carrier appetite, HOA delinquency rates, flood-zone X vs. AE, foundation cracks on 1960s–1980s slab homes, and the school-zone resale math that catches Tier A buyers off-guard. Reusable; not personalized.
4. **The 3-Lender Comparison Card.** Three lenders the agent has personally worked with, named, with a one-line "best for" tag (e.g., *"best for credit in the 640–680 range,"* *"fastest close,"* *"FHA / 3.5% down specialist"*). The agent does not receive a referral fee from any of them. The prospect leaves with phone numbers and emails, not a "I'll have someone reach out."

### Guarantee (risk reversal)

> **The 30-Minute Promise.**
>
> If after our 30 minutes you don't walk away with (1) a written buy-box, (2) a personalized 3-neighborhood shortlist, and (3) a clearer next step than you had when you booked — I'll send you the shortlist anyway and personally introduce you to one of the three lenders for a free pre-approval call. No commitment to use me as your agent. Ever.

The guarantee is operationally cheap (the deliverables are produced regardless) and removes the only real fear: *"What if I waste 30 minutes and it's just a pitch?"* Stating it explicitly on the booking page is what makes it work.

### Call to action

Single-button on every CTA surface (LM1 Tier A result page, LM2 viewer page, BSS landing page, every Tier A nurture email):

> **Book My 30-Minute Buyer Strategy Session**

No alternate CTAs on the same page. No "or call me directly." One door.

---

## Eligibility — who books

The BSS is **only** offered to prospects who have crossed one of these thresholds:

| Source | How it triggers | Pipedrive `fthb_bss_source` |
|---|---|---|
| LM1 Tier A result | CTA on the `READY_NOW` result page; CTA in Campaign **FTHB LM1 - Tier A** emails A1–A5 | `fthb_lm1_tier_a` |
| LM2 viewer (any tier originally) | CTA on the rendered roadmap page and in the LM2 transactional email, *only when the contact is not currently in Tier B nurture* | `fthb_lm2_viewer` |
| Warm outreach 1-on-1 | Agent shares the booking link directly in a DM, text, or email after a warm conversation | `fthb_warm_outreach` |
| Referral | Existing client or sphere contact sends someone to the booking link | `fthb_referral` |

**Hard exclusion: Tier C (`FOUNDATION`) prospects are never offered the BSS.** This is enforced in three places:

1. The Tier C result page CTA is the monthly market update, not the BSS.
2. The **FTHB LM1 - Tier C** Campaign never contains a BSS link.
3. The Pipedrive Workflow Automation that watches for BSS bookings checks `fthb_lm1_tier` — if it's `FOUNDATION` and the contact somehow booked anyway (e.g., via a Tier A booking link forwarded by a friend), the automation fires an internal Slack notification to the agent so the agent can decide manually. It does not auto-cancel.

Re-takes change tier. If a prospect re-takes LM1 and moves from Tier C to Tier B or A, the new tier governs and the eligibility above kicks in.

---

## 30-minute run-of-show

The block-by-block agenda for the call. Times are calibration targets, not contracts — flex as the conversation requires, but never run long past 30 minutes without explicit consent ("we have five more minutes — keep going, or send the rest in writing?"). Respecting the time cap is part of the deal.

| Block | Minutes | What happens | Why this block exists |
|---|---|---|---|
| **Welcome + frame** | 0:00–3:00 | Agent reads the scorecard answers back in one sentence. Frames the 30 minutes (buy-box, shortlist, next step, no paperwork today). Then asks the branching question: *"Before I show you neighborhoods, two ways we can do this. Either you tell me where you're already looking and I'll run the math on those places first, or I show you three I'd pick from your numbers and you tell me what's missing. Which one?"* The answer routes Block 4 down Path A (no preference), Path B (preference + math works), or Path C (preference + math does not work). Full script in [10-bss-content.md](10-bss-content.md). | Sets the diagnostician frame, removes the "is this going to be a pitch?" tension, and gives the prospect control of the shortlist agenda on minute one. |
| **Diagnostic (gap-filling)** | 3:00–10:00 | Agent asks the 4–5 questions LM1 didn't cover: down-payment source (gift / savings / 401k), household composition + commute reality, what they're *unwilling* to compromise on, what their backup plan is if they don't buy in 6 months. | LM1's 10 questions are calibrated for tier scoring, not for buy-box construction. These five fill the gaps. |
| **Live math** | 10:00–20:00 | Agent screen-shares the Google Sheet bonus. Plugs in the prospect's numbers in real time: price ceiling at today's rate, monthly payment, closing costs, reserve cushion. Toggles FHA vs. conventional. Adjusts down payment. | This is the single highest-impact moment. Watching the math live in their own scenario reframes "can I afford this?" from anxiety to arithmetic. |
| **Shortlist walk-through** | 20:00–27:00 | Branches on the answer from Block 1. **Path A** (no preference): agent presents three ZIPs along the Sanford to Downtown Orlando route picked against the math: an anchor, a stretch, and a wildcard. **Path B** (preference + math works): agent runs the math on the prospect's named ZIPs first and adds one agent-picked alternate at the end; prospect's preferences always lead the list. **Path C** (preference + math does not work): agent shows the gap honestly on the named ZIP, then surfaces three levers (wait and grow cash, downsize within the ZIP, shift one ZIP outward). The shortlist that ships in the PDF always shows a path, never a "you cannot afford this." Decision rules and copy in [12-bss-math-and-shortlist.md](12-bss-math-and-shortlist.md). | Replaces a month of Zillow with a 7-minute curated walkthrough by someone who has actually shown houses on these streets, while preserving prospect agency over the search. |
| **Next step + close** | 27:00–30:00 | Agent: *"Three options for what happens next. Option 1: I send you the shortlist PDF in 24 hours and you take it from here — no follow-up from me. Option 2: I introduce you to one of the lenders we discussed for a free pre-approval, and you decide after that. Option 3: I become your buyer's agent and we go tour the top two next weekend. Pick whichever fits. Either way, the shortlist is yours."* | Mirrors the A5 "either way works for me" close. Three options is enough to be useful, few enough to feel decisive. |

The agent walks into every BSS with the prospect's LM1 scorecard tab open and the Google Sheet template pre-loaded with empty input cells. **Calibrate the timing during the first 5–10 BSSes**; whichever block consistently runs over or under is the one to retune in `src/config/bss.ts`.

---

## Pre-call operations

Triggered automatically when a calendar booking arrives.

### Intake form

3 minutes. Sent in the booking confirmation email and as a link in the calendar invite. Submission is optional but heavily nudged ("the more I know, the less of our 30 minutes we spend on warm-up").

| Field | Type | Required | Why |
|---|---|---|---|
| Partner / co-buyer? | bool + name field if yes | yes | If yes, they should be on the call |
| Target move-in window | enum: `0–3 mo` / `3–6 mo` / `6–12 mo` / `12+ mo` | yes | Calibrates urgency and which lender to suggest |
| Down payment source | multi-select: `savings` / `gift` / `401k` / `home sale` / `not sure yet` | yes | Drives the math block — gift funds and 401k withdrawals have specific gotchas |
| Approximate down payment available | enum bucket: `<$10k` / `$10–25k` / `$25–50k` / `$50k+` | yes | Calibrates the price ceiling math |
| Neighborhoods already considering | free text, optional | no | Load-bearing for the shortlist branch (Path A / B / C in [12-bss-math-and-shortlist.md](12-bss-math-and-shortlist.md)). Pre-loads the math sheet so the agent can run the math on the prospect's named ZIPs first instead of leading with his own picks. Blank means the agent brings three. |
| Top 2 must-haves | free text, 2 fields | no | Drives the shortlist (e.g., "garage," "yard for dog," "under 30 min to UCF") |
| Hard deal-breakers | free text | no | Same |
| Anything else you want me to know before the call? | free text, optional | no | Catches the things the buckets miss |

Form storage: posts to the same Make.com webhook with `magnet: "fthb_bss_intake"`. Make.com writes to Pipedrive against the existing contact (matched by email) and updates `fthb_bss_intake_completed_at`.

### Calendar booking flow

1. Prospect clicks **Book My 30-Minute Buyer Strategy Session** on any eligible CTA surface. Source tag (`fthb_lm1_tier_a` / `fthb_lm2_viewer` / etc.) passes through as a hidden field on the booking link.
2. Cal.com (or equivalent) collects: first name, email, scheduled time, and the source tag. **Phone is not collected at booking** — the call is video by default, and asking for a phone number raises friction without proportional value. If the prospect prefers phone, they tell the agent on the intake form.
3. Cal.com fires a webhook to Make.com on booking. Make.com:
   - Looks up the contact in Pipedrive by email.
   - Creates the contact if missing (warm outreach / referral entrants).
   - Sets `fthb_bss_booked_at`, `fthb_bss_scheduled_for`, `fthb_bss_source`.
4. A Pipedrive Workflow Automation watches for `fthb_bss_booked_at` getting set and triggers the **FTHB BSS - Booking Confirmation** Campaign (BSS-1).
5. A second Workflow Automation pauses the contact's currently-active LM1 or LM2 nurture Campaign so they don't get a nurture email between booking and the call.

### Pre-call email touchpoints

All sent by Pipedrive Campaigns (not Make.com). Calibrate cadence against real no-show rates during Phase 4.

| ID | Trigger | Subject (proposed) | Purpose |
|---|---|---|---|
| **BSS-1** | Immediately on booking | *"You're booked — here's the 3-minute prep"* | Confirmation, calendar add, intake form link, what to expect on the call |
| **BSS-2** | 24 hours before the call | *"Tomorrow at {{time}} — one quick thing"* | Reminder, video link, intake form nudge if `fthb_bss_intake_completed_at` is null |
| **BSS-3** | 1 hour before the call | *"In an hour — see you on the video link"* | Final reminder, video link only |

If the intake form is still incomplete at the 24-hour mark, BSS-2 includes a direct ask for the three most-load-bearing answers (down payment bucket, timeline, partner Y/N) so the call isn't blind even without the form.

---

## Post-call operations

The call ends. Within 24 hours, the deliverables go out. Within 7 days, the prospect is back on a Campaign appropriate to their outcome.

### Deliverable turnaround (manual, but scripted)

| Artifact | Window | How |
|---|---|---|
| Personalized 3-Neighborhood Shortlist PDF | 24 hours | Agent fills a Notion / Google Doc template with the three ZIPs, comps, and trade-offs from the call. Exports to PDF. Sends via BSS-4. |
| Filled-in Payment & Closing-Cost Sheet | End of call (already shared live) | The link is already in the prospect's hands. No follow-up action required. |
| Lender intro | 48 hours, **only if the prospect chose Option 2 or 3** | Agent emails the chosen lender with the prospect copied. One-paragraph intro: tier, timeline, what to focus on, what the prospect already knows. |

### Outcome recording

Within 1 hour of the call ending, the agent sets `fthb_bss_outcome` in Pipedrive to one of:

| Value | Means |
|---|---|
| `signed_buyer_agreement` | Prospect chose Option 3 and signed the buyer's representation agreement on or shortly after the call |
| `lender_intro_made` | Prospect chose Option 2; agent has made (or will make within 48 hours) the lender introduction |
| `shortlist_only` | Prospect chose Option 1; deliverables sent, no further commitment |
| `not_ready_yet` | Prospect attended but the conversation surfaced that they're actually Tier B or Tier C in disguise (e.g., down payment is thinner than the LM1 scorecard captured); re-enroll in appropriate nurture |
| `no_show` | Prospect did not attend and did not reschedule |

### Post-call email + Campaign routing

A Pipedrive Workflow Automation watches `fthb_bss_outcome` and routes accordingly:

| Outcome | What fires |
|---|---|
| `signed_buyer_agreement` | **FTHB BSS - Active Client Onboarding** Campaign begins (out of scope of this doc — that's the client-side workflow) |
| `lender_intro_made` | **BSS-4 Deliverables** sent (shortlist PDF + lender intro confirmation). Day 7 check-in (**BSS-5**) sent. Day 21 re-enter monthly market update list. |
| `shortlist_only` | **BSS-4 Deliverables** sent (shortlist PDF). Day 7 check-in (**BSS-5**) sent. Day 21 re-enter monthly market update list. |
| `not_ready_yet` | Re-enroll in **FTHB LM1 - Tier B** or **FTHB LM1 - Tier C** based on what the call revealed; update `fthb_lm1_tier` to match. Send **BSS-6 (gentle redirect)** explaining the re-routing in the agent's voice. |
| `no_show` | **BSS-NS-1** (no-show reschedule offer) fires 1 hour after the missed slot. If no reschedule within 7 days, **BSS-NS-2** fires once. After that, contact returns to whatever Campaign they were paused from. No third attempt. |

### Post-call email templates

All sent by Pipedrive Campaigns. Drafts to be authored in `email-drafts/bss/` mirroring the structure of `email-drafts/lm1-readiness-filter/`.

| ID | Subject (proposed) | Trigger | Goal |
|---|---|---|---|
| **BSS-4** | *"Your shortlist + numbers, as promised"* | `fthb_bss_outcome` set to `shortlist_only` or `lender_intro_made` | Deliver the PDF, recap the call, restate the three options |
| **BSS-5** | *"Quick check-in — anything come up?"* | Day 7 after BSS-4 | Open the door for a question without pitching |
| **BSS-6** | *"Reset — here's where I'd actually start"* | `fthb_bss_outcome = not_ready_yet` | Frame the re-routing as a favor, not a downgrade; restart their tier-appropriate nurture |
| **BSS-NS-1** | *"Missed you — pick another time?"* | 1 hour after a no-show | One-click reschedule link, no guilt |
| **BSS-NS-2** | *"Last one from me on the strategy session"* | 7 days after BSS-NS-1 if no rebooking | Sign-off mirroring A5's tone — open door, no pressure |

---

## Pipedrive custom fields (namespace: `fthb_bss_*`)

Created on the Person object, alongside the existing `fthb_lm1_*` and `fthb_lm2_*` fields from [02-readiness-filter-spec.md](02-readiness-filter-spec.md) and [05-process-map-spec.md](05-process-map-spec.md). All field names use the `fthb_` cross-system prefix per [CLAUDE.md](../CLAUDE.md)'s namespace convention.

| Field | Type | Set by | Notes |
|---|---|---|---|
| `fthb_bss_booked_at` | datetime | Make.com on Cal.com webhook | First-time-set fires the BSS-1 Campaign |
| `fthb_bss_scheduled_for` | datetime | Make.com on Cal.com webhook | Drives BSS-2 (T-24h) and BSS-3 (T-1h) Campaign timing |
| `fthb_bss_source` | enum | Make.com on Cal.com webhook | `fthb_lm1_tier_a` / `fthb_lm2_viewer` / `fthb_warm_outreach` / `fthb_referral` |
| `fthb_bss_intake_completed_at` | datetime, nullable | Intake form via Make.com | Drives the nudge logic in BSS-2 |
| `fthb_bss_attended_at` | datetime, nullable | Agent (manual) within 1 hour of call end | Empty → no-show flow |
| `fthb_bss_outcome` | enum | Agent (manual) within 1 hour of call end | Drives all post-call routing |
| `fthb_bss_followup_sent_at` | datetime, nullable | Pipedrive Workflow Automation after BSS-4 fires | Audit field for "did the deliverables actually go out" |
| `fthb_bss_rescheduled_count` | integer, default 0 | Pipedrive Workflow Automation on each reschedule | Cap at 2 reschedules before the slot is offered manually only |

The pre-call intake form's free-text fields (must-haves, deal-breakers, anything else) are stored on the Pipedrive Person as notes / a single concatenated text custom field rather than separate columns — they're for the agent's call prep, not for routing.

---

## Analytics events (namespace: `fthb_bss_*`)

Fired client-side from the BSS landing page and the Cal.com embed. Wire alongside the existing `fthb_readiness_*` and `fthb_roadmap_*` events from [02-readiness-filter-spec.md](02-readiness-filter-spec.md) and [05-process-map-spec.md](05-process-map-spec.md). All event names use the `fthb_` cross-system prefix.

| Event | When |
|---|---|
| `fthb_bss_landing_view` | The BSS landing page renders (whether arrived from LM1 Tier A, LM2 viewer page, or a direct link) |
| `fthb_bss_cta_click` | The "Book My 30-Minute Buyer Strategy Session" button is clicked, regardless of surface |
| `fthb_bss_calendar_view` | The Cal.com embed renders (proxy for "got past the landing page") |
| `fthb_bss_booked` | Cal.com booking confirmation page renders (also redundantly fired by Make.com server-side; keep both for funnel debugging) |
| `fthb_bss_intake_submit` | Intake form successful POST |
| `fthb_bss_attended` | Fired by a Pipedrive Workflow Automation when `fthb_bss_attended_at` is set (server-side event surfaced to analytics via Plausible/PostHog ingest URL) |
| `fthb_bss_outcome_recorded` | Same trigger as above but fires once `fthb_bss_outcome` is set; outcome value is the event property |

The key funnel ratios to watch in Phase 4: `landing_view → cta_click`, `cta_click → booked`, `booked → attended`, `attended → signed_buyer_agreement | lender_intro_made`. Diagnose the worst-performing step first; do not redesign the whole funnel against a single ratio.

---

## Make.com webhook payloads

Two BSS-related payloads share the existing Make.com webhook URL, distinguished by `magnet`.

### `magnet: "fthb_bss_intake"` (from the intake form)

```jsonc
{
  "magnet": "fthb_bss_intake",
  "source": "fthb_lm1_tier_a", // or fthb_lm2_viewer / fthb_warm_outreach / fthb_referral
  "contact": {
    "email": "maria@example.com",
    "first_name": "Maria"
  },
  "intake": {
    "co_buyer": { "present": true, "name": "Carlos" },
    "timeline": "3-6 mo",
    "down_payment_source": ["savings", "gift"],
    "down_payment_bucket": "$25-50k",
    "neighborhoods_considering": "Sanford and maybe Lake Mary",
    "must_haves": ["garage", "under 25 min to Maitland"],
    "deal_breakers": "no flood zone AE",
    "notes": "Relocating from San Juan in August"
  }
}
```

Make.com action: match the Pipedrive Person by email; if missing, create. Set `fthb_bss_intake_completed_at = now()`. Append the free-text fields to the Person's notes.

### `magnet: "fthb_bss_booking"` (from the Cal.com webhook)

```jsonc
{
  "magnet": "fthb_bss_booking",
  "source": "fthb_lm1_tier_a", // hidden field on the Cal.com booking link
  "contact": {
    "email": "maria@example.com",
    "first_name": "Maria"
  },
  "booking": {
    "scheduled_for": "2026-05-24T14:30:00-04:00",
    "booked_at": "2026-05-17T09:12:00-04:00",
    "video_link": "https://meet.example.com/abc123"
  }
}
```

Make.com action: match or create the Pipedrive Person; set `fthb_bss_booked_at`, `fthb_bss_scheduled_for`, `fthb_bss_source`. The Workflow Automation watching `fthb_bss_booked_at` does the rest.

Both payloads write a row to the existing Google Sheet log alongside LM1 and LM2 submissions for the same reason: if Pipedrive sync fails, the row is still recoverable.

---

## Pipedrive Workflow Automations summary

For the build-side reference. None of this logic lives in Make.com.

| Automation | Trigger | Action |
|---|---|---|
| **BSS Booking Confirmation** | `fthb_bss_booked_at` set | Enroll in **FTHB BSS - Booking Confirmation** Campaign (sends BSS-1, BSS-2, BSS-3) |
| **Pause Nurture During BSS Window** | `fthb_bss_booked_at` set | Unenroll from any active **FTHB LM1 - Tier *** or **FTHB LM2 - Roadmap** Campaign until 7 days after `fthb_bss_scheduled_for` |
| **Post-call Routing** | `fthb_bss_outcome` set | Branch on outcome value; enroll/re-enroll in the appropriate Campaign per the table in "Post-call email + Campaign routing" |
| **Tier C Guardrail** | `fthb_bss_booked_at` set AND `fthb_lm1_tier == FOUNDATION` | Fire internal Slack notification to the agent; do not auto-cancel |
| **No-Show Handling** | `fthb_bss_scheduled_for < now()` AND `fthb_bss_attended_at` null | Enroll in **FTHB BSS - No-Show Reschedule** Campaign |
| **Reschedule Cap** | `fthb_bss_rescheduled_count >= 2` | Disable the booking link for this contact; agent reschedules manually only |

---

## What lives in `src/config/bss.ts`

Per the [CLAUDE.md](../CLAUDE.md) configuration principle, anything calibration-sensitive lives in config so Phase 4 tuning is a config edit, not a refactor.

```ts
// Sketch — not final. Confirm shape against the rest of src/config/ once the
// host site audit (requirements/00-foundations/01-audit-existing-site.md) is done.
export const BSS_CONFIG = {
  durationMinutes: 30,
  intakeNudgeAtHoursBefore: 24,
  finalReminderAtHoursBefore: 1,
  followUpSlaHours: 24,         // shortlist PDF deadline after call ends
  lenderIntroSlaHours: 48,
  day7CheckInAfterDays: 7,
  monthlyListReentryAfterDays: 21,
  rescheduleMax: 2,
  noShowFollowUps: 2,           // BSS-NS-1, then BSS-NS-2, then stop
} as const;
```

Structural constants (the five outcome enum values, the three CTA option labels) can stay inline; they're not knobs.

---

## Voice and copy rules

All BSS-facing copy — landing page, calendar tool description, intake form, BSS-1 through BSS-NS-2 emails, the in-call script — follows the project rules from [CLAUDE.md](../CLAUDE.md). Specifically:

- **Diagnostician, not salesperson.** The BSS landing page must show what's *in* the call before it shows the booking button. The reader should feel they already know what the 30 minutes will contain.
- **Either way works for me.** The Option 1 / Option 2 / Option 3 close is the BSS in microcosm. Mirror this language in BSS-4 and BSS-5 — never escalate pressure after the call ends.
- **No "Free Buyer's Guide" energy.** Vague freebies are explicitly banned. The BSS landing page names the specific result (buy-box + 3-neighborhood shortlist + next step) and the specific time-to-result (30 minutes).

---

## Hard rules (do not violate)

Inherited from [CLAUDE.md](../CLAUDE.md) and made specific to the BSS:

1. **Tier C is never offered the BSS.** Enforced by the Pipedrive automation, the result-page CTA logic, and the absence of a BSS link in any Tier C Campaign. If a Tier C contact lands on the BSS landing page via a forwarded link, the page itself does not block them — but the Tier C Guardrail automation flags it to the agent for manual review.
2. **No paid ads or boosted posts driving BSS bookings.** Every booked BSS comes from organic content, LM1, LM2, warm outreach, or referral. No exceptions.
3. **No cold outreach to fill the calendar.** The agent does not buy lists, scrape, or DM strangers to pad the BSS pipeline.
4. **No third lead magnet.** The BSS is the offer the two lead magnets terminate at, not a third magnet of its own. The BSS landing page exists, but it is not promoted as a magnet — it is the booking surface for prospects who have already qualified themselves.
5. **30 minutes means 30 minutes.** The cap is part of the value equation (Effort & Sacrifice). Running 45-minute calls trains the audience to expect them, and the math stops working.

---

## What's intentionally out of scope of this doc

- The buyer's representation agreement itself (post-`signed_buyer_agreement` workflow). That's the active-client onboarding spec — out of scope.
- The exact wording of BSS-1 through BSS-NS-2 (subject lines are proposed here; full email drafts belong in `email-drafts/bss/` mirroring the existing LM1/LM2 structure).
- The Shortlist PDF template (Notion vs. Google Doc vs. typed Astro page exported via headless Chrome). Pick during build.
- The exact analytics tooling (Plausible vs. PostHog); both can fire the events listed above.
- The Google Sheet template for the live-math bonus. The structure is implied by the field list under "Live math" in the run-of-show but the actual sheet design is a build deliverable.
- Spanish-language versions of the landing page, intake form, in-call script, post-call emails, and shortlist PDF. Deferred to a later iteration; the first launch is English only. When Spanish ships, restore: the `fthb_bss_language` Pipedrive field, the language field on the intake form, the language key on the `fthb_bss_intake` payload, and a "delivered in your language" line in the bonus stack.

---

## Open questions to resolve before locking

- **Lock the public name + subhead.** The proposed name above must be tested against the Hormozi voice rules and locked verbatim before any landing page copy goes live.
- **Calibrate the deliverable SLAs.** 24 hours for the shortlist PDF and 48 hours for the lender intro are starting points. If the agent can credibly hit 4 hours and 12 hours, say so on the landing page — faster turnaround compresses the time-delay term in the value equation directly.
- **Calendar tool selection.** Cal.com is the default per [08-implementation-roadmap.md](08-implementation-roadmap.md), but the choice is reversible. Decide before building the webhook integration.
- **No-show retry policy.** Two follow-ups (BSS-NS-1, BSS-NS-2) and then stop is the proposed cap; test against real data in Phase 4.
- **Re-take re-eligibility.** If a contact re-takes LM1 and moves Tier C → Tier B → Tier A across several months, when do they become BSS-eligible? Proposed: instantly on the first Tier A or Tier B (via LM2) result, regardless of prior history. Confirm.

---

## Related documents

- [01-strategy-and-funnel.md](01-strategy-and-funnel.md) — Why the BSS exists and how it sits at the end of the funnel
- [02-readiness-filter-spec.md](02-readiness-filter-spec.md) — LM1 spec; defines Tier A as the primary BSS entry
- [04-readiness-filter-emails.md](04-readiness-filter-emails.md) — Tier A emails A1–A5 are the primary email surface for the BSS CTA
- [05-process-map-spec.md](05-process-map-spec.md) — LM2 spec; defines the secondary BSS entry
- [07-process-map-emails.md](07-process-map-emails.md) — LM2 emails carry the BSS CTA for warmed Tier B graduates
- [08-implementation-roadmap.md](08-implementation-roadmap.md) — Phased build plan; BSS infrastructure (calendar, Pipedrive fields, Make.com routing) lands in Phase 0–1
- [10-bss-content.md](10-bss-content.md) — Landing page, intake form copy, in-call script, deliverable templates
- [11-bss-emails.md](11-bss-emails.md) — Pre-call, post-call, and no-show email drafts
- [12-bss-math-and-shortlist.md](12-bss-math-and-shortlist.md) — Math sheet structure, formulas, shortlist decision logic, own-zip handling
- [CLAUDE.md](../CLAUDE.md) — Funnel namespace convention, hard rules, voice
