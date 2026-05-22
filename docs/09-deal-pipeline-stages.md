# Deal Pipeline Stages — Pipedrive

This doc defines the Pipedrive **Deal pipeline** for the FTHB funnel: the six stages a contact passes through from quiz submission to closing day, the probability and rotting values for each, the rules for how Deals move between stages, and the Make.com logic that sets stages automatically.

## Scope

This document covers the **Deal** pipeline only. In the FTHB funnel's iteration-1 architecture:

- **Tier A, Tier B, and Roadmap contacts are Deals** — they pass through this pipeline.
- **Tier C contacts are Leads** — they live outside this pipeline, in the Lead pool, enrolled in the `FTHB LM1 - Tier C` automation. There's no progressive pipeline for Leads; they either complete the Tier C automation and get manually reviewed (typical action: add to monthly newsletter) or they get promoted to a Deal by a later event (downloading LM2, or a retake to Tier A/B), at which point they enter this pipeline.

For the full Person / Lead / Deal entity model and the Make.com conflict-resolution rules that decide which entity to create or update, see [`CLAUDE.md`](../CLAUDE.md) → "Person vs. Lead vs. Deal split" and [`04-readiness-filter-emails-v2.md`](04-readiness-filter-emails-v2.md) → "Make.com routing logic."

## The six stages — quick reference

| # | Stage | Probability | Rotting | Description |
|---|---|---|---|---|
| 1 | **Quiz Taken** | 10% | 14 days | Tier A or B contact who just submitted the LM1 scorecard |
| 2 | **Received Roadmap** | 20% | 65 days | Downloaded LM2 (the 9-Step Roadmap); deeper engagement |
| 3 | **BSS Booked** | 40% | 14 days | Buyer Strategy Session scheduled in Google Calendar |
| 4 | **Pre-Approved & Searching** | 65% | 30 days | Real lender pre-approval letter in hand; touring properties |
| 5 | **Under Contract** | 80% | 14 days | Offer accepted; inspection / appraisal / financing contingencies still open |
| 6 | **Closing** | 95% | 30 days | Contingencies cleared, lender's "clear to close" issued; awaiting closing date |

After stage 6, Deals close through Pipedrive's built-in **Won** / **Lost** status — no separate stage needed.

---

## Stage details

### 1. Quiz Taken

**Probability: 10% · Rotting: 14 days**

A contact has just submitted the LM1 readiness scorecard and scored into Tier A (`READY_NOW`) or Tier B (`NINETY_DAY`). They're brand new to the pipeline. Make.com has created the Deal and enrolled it in the corresponding tier automation (`FTHB LM1 - Tier A` or `FTHB LM1 - Tier B`).

**Who's at this stage:** A Tier B contact who hasn't downloaded the Roadmap yet. Or a Tier A contact who hasn't booked the BSS yet. Either way, they've engaged with the funnel exactly once.

**Why 10%:** A scorecard submission is a real interest signal but a small one. Buy-side real estate funnels typically convert 5–15% of top-funnel leads to closed deals over an 18-month window. 10% sits in the middle of that band.

**Why 14 days rotting:** The Tier A automation is 5 emails over 14 days; the Tier B automation is 3 emails over 5 days followed by 9+ days of silence. By day 14, the automation has run its course. If nothing has moved by then (no Roadmap download, no BSS booking), the funnel has done its job and the agent should manually review.

**Agent actions at this stage:** mostly hands-off — the tier automation is doing the work. The agent's primary action is responding to inbound replies (the email copy explicitly invites them).

**Exits:**
- → Received Roadmap (LM2 download — Make.com moves automatically)
- → BSS Booked (manual, on calendar booking)
- → Closed Lost (after manual review past rotting, if cold)

### 2. Received Roadmap

**Probability: 20% · Rotting: 65 days**

The contact has downloaded LM2 (the 9-Step First Home Roadmap). They're enrolled in the `FTHB LM2 - Roadmap` automation (9 nurture emails over 58 days, plus the transactional delivery email). The Roadmap is the deepest free asset in the funnel; downloading it is a meaningful engagement signal beyond the quiz.

**Who's at this stage:** A Tier B contact who clicked through and grabbed the Roadmap (most common). A standalone-Roadmap visitor who came in via `/orlando-homebuying-roadmap` without taking the quiz first. A Tier C Lead who downloaded the Roadmap and got promoted to a Deal in the process.

**Why 20%:** Roughly 2× the Quiz Taken rate. Roadmap downloaders show more intent because they've actively pursued a deeper free resource and signaled they want process-level help, not just a score. Industry data on lead-magnet-stacked funnels supports ~15–25% close rates for second-tier engagers.

**Why 65 days rotting:** The Roadmap automation is 58 days end-to-end (a Day 2 opener followed by 8 emails on a weekly cadence, finishing with the direct BSS ask on Day 58), plus a 7-day buffer for the contact to respond after the final email. If nothing has happened by day 65 (no BSS booked, no reply, no movement), the funnel has fully run and the agent should review.

**Agent actions:** Watch for replies — the Roadmap N2 email invites replies and is the highest-yield reply email in the funnel. Be ready to send 2–3 lender referrals on request.

**Exits:**
- → BSS Booked (manual)
- → Pre-Approved & Searching (rare — manual move if the contact gets pre-approved without booking a BSS first)
- → Closed Lost (manual review past rotting)

### 3. BSS Booked

**Probability: 40% · Rotting: 14 days**

The contact has booked a Buyer Strategy Session on the agent's Google Calendar. The agent has manually moved the Deal to this stage (Google Calendar → Pipedrive is not automated; see [`CLAUDE.md`](../CLAUDE.md) "Hard tooling constraints" for the rationale).

**Why 40%:** A booked call is a major intent signal. Real estate lead-to-call conversion data puts booked-call-to-close rates in the 35–50% range. 40% is a conservative middle.

**Why 14 days rotting:** A BSS that hasn't happened within 2 weeks is either a no-show, a reschedule the agent forgot to track, or a deal that's drifting. Two weeks is enough buffer for one reschedule.

**Agent actions:** Hold the BSS. After the call, manually update Pipedrive: move to **Pre-Approved & Searching** if the contact has (or commits to getting) a pre-approval letter and is actively going to tour homes. Move to **Closed Lost** if the call revealed they're not the right fit. Or leave at BSS Booked if a follow-up call is scheduled.

**Exits:**
- → Pre-Approved & Searching (manual, after the call)
- → Closed Lost (manual, if the call revealed misalignment)

### 4. Pre-Approved & Searching

**Probability: 65% · Rotting: 30 days**

The contact has a real pre-approval letter (not pre-qualification) from a lender and is actively touring properties. The agent confirmed both during or after the BSS call.

**Why 65%:** Pre-approval is the single biggest qualifier in residential real estate. Pre-approved active buyers close 55–75% of the time within 6 months in normal markets.

**Why 30 days rotting:** Active searches take time — most first-time buyers see 8–15 homes before making an offer, and that can stretch 4–8 weeks. 30 days rotting means "check in monthly to make sure they haven't gone dark," not "deal is dead."

**Agent actions:** This is the high-touch phase. Property recommendations, showings, market data, offer strategy. The bulk of the agent's time per deal happens here.

**Exits:**
- → Under Contract (manual, on offer acceptance)
- → Closed Lost (manual — buyer paused, lost confidence, switched plans, etc.)

### 5. Under Contract

**Probability: 80% · Rotting: 14 days**

An offer has been accepted. The deal is now in the contingency window — typically 10–15 days in Florida for inspection, plus the financing and appraisal contingencies running in parallel. This is where most deals that fall through fall through.

**Why 80%:** National fall-through rates for purchase contracts run 10–15%; FL is roughly in line. Of every 10 deals that go under contract, 8–9 close. The 20% gap from "100% closing" lives almost entirely in this stage — inspection findings, appraisal gaps, financing snags.

**Why 14 days rotting:** FL inspection periods are typically 10–15 days. If the deal is still in Under Contract past 14 days, contingencies haven't been cleared and something is stuck — re-negotiating after inspection, appraisal came in low, lender re-doc request, etc.

**Agent actions:** Coordinate inspection, advocate during contingency negotiations, monitor lender milestones. When the lender issues the "clear to close" (CTC), move the deal to **Closing**.

**Exits:**
- → Closing (manual, on lender's CTC)
- → Closed Lost (manual, if the deal terminates during contingencies)

### 6. Closing

**Probability: 95% · Rotting: 30 days**

Contingencies cleared, financing locked, lender has issued the "clear to close" notification. The deal is essentially done — just waiting for the calendar date.

**Why 95%:** Clear-to-close deals close ~95–97% of the time. The remaining 3–5% is last-minute fraud flags, sudden buyer-credit changes, or rare title issues.

**Why 30 days rotting:** A typical FL closing happens 30–45 days from contract acceptance; subtract the 10–15 days in Under Contract and the Closing phase is usually 15–30 days. 30 days rotting flags closings that are running long.

**Agent actions:** Coordinate the final walkthrough, attend the closing, hand over keys. After closing, mark the Deal **Won** with closing date and final purchase price.

**Exits:**
- → Closed Won (manual, on closing day)
- → Closed Lost (rare — last-minute fall-through)

---

## Movement chain

A Deal moves forward through stages — never backward. The Make.com layer enforces this on the automated transitions (see "Make.com stage-setting logic" below); the agent enforces it on the manual ones.

```
[entry]
   │
   ▼
Quiz Taken ────────────────┐                          (automated)
   │                       │
   ▼                       ▼
Received Roadmap ←─── [LM2 download for any prior stage Deal that hasn't passed this point]
   │
   ▼
BSS Booked              (manual — Google Calendar booking)
   │
   ▼
Pre-Approved & Searching (manual — after BSS)
   │
   ▼
Under Contract          (manual — offer accepted)
   │
   ▼
Closing                 (manual — lender's clear-to-close)
   │
   ▼
Closed Won  /  Closed Lost  (manual)
```

### Why "forward only"

Suppose a contact at **Pre-Approved & Searching** retakes the LM1 quiz (maybe to update their answers). Make.com would otherwise try to drop them at Quiz Taken — but that would erase real pipeline progress. The rule: Make.com only **advances** a Deal's stage, never **regresses** it. If the existing stage is later than where the webhook would place the contact, update the contact's fields (tier, score, retake timestamp) but leave the stage alone.

This rule lives in Make.com config and is the load-bearing piece of "monotonic stage progression."

---

## How contacts enter the pipeline

There are four practical paths.

### Path A — LM1 Tier A (the BSS-direct path)

```
LM1 webhook (tier = READY_NOW)
   │
   ▼
Make.com: create Deal (or update existing) → stage = Quiz Taken
   │
   ▼
Enroll in FTHB LM1 - Tier A automation
```

The contact stays at Quiz Taken until they book a BSS (manual move) or the Deal rots (manual review).

### Path B — LM1 Tier B → Roadmap (the primary path)

```
LM1 webhook (tier = NINETY_DAY)
   │
   ▼
Make.com: create Deal → stage = Quiz Taken
   │
   ▼
Enroll in FTHB LM1 - Tier B automation (3 emails / 5 days, pushes LM2)
   │
   ▼ (contact downloads LM2 — typically mid-sprint)
   ▼
LM2 webhook
   │
   ▼
Make.com: fire transactional (delivers PDF) + advance Deal stage to Received Roadmap
(long Roadmap nurture enrollment is deferred — Tier B continues to completion)
   │
   ▼ (Tier B sprint completes; final step writes fthb_tier_b_completed_at)
   ▼
Tier B completion trigger
   │
   ▼
Make.com: if fthb_received_lm2 = true → enroll in FTHB LM2 - Roadmap (58-day nurture)
```

Tier B and the long Roadmap nurture run as a **sequential handoff**, not in parallel. The LM2 webhook delivers the Roadmap immediately (via the transactional) and advances the Deal stage so pipeline progress reflects the engagement, but the long Roadmap nurture (`FTHB LM2 - Roadmap`) only enrolls after the Tier B sprint completes. A Tier B contact who never downloads LM2 finishes the sprint and drops out of active nurture. See the Tier B completion trigger in the Make.com stage-setting logic below.

### Path C — Roadmap standalone (no prior quiz)

```
LM2 webhook (source = fthb_lm2_standalone, no prior Deal/Lead)
   │
   ▼
Make.com: create new Deal → stage = Received Roadmap
   │
   ▼
Enroll in FTHB LM2 - Roadmap automation
```

If the contact later takes the LM1 quiz and lands in Tier A or B, the existing Deal is updated (fields refreshed, stage left at Received Roadmap or wherever it has progressed to).

### Path D — Tier C Lead → Roadmap (promotion path)

```
Existing Tier C Lead in FTHB LM1 - Tier C automation
   │
   ▼ (contact downloads LM2 via standalone landing page)
   ▼
LM2 webhook
   │
   ▼
Make.com: convert Lead → Deal (copy custom fields) → new Deal stage = Received Roadmap
   │
   ▼
Enroll new Deal in FTHB LM2 - Roadmap automation
```

The Tier C automation enrollment is left behind on the converted Lead record. Iteration 1 accepts losing the remaining Tier C drip — the Roadmap content is more relevant to a contact who actively engaged at a deeper level.

---

## Make.com stage-setting logic

Make.com sets the Deal stage during the LM1 and LM2 scenarios. The rules:

### LM1 webhook (tier A or B)

```
If no existing Deal:
   Create Deal at stage = Quiz Taken
Else (existing Deal):
   If current stage is Quiz Taken or earlier:
      No-op on stage (already there)
   Else:
      No-op on stage (forward-only rule — don't regress)
   In both cases: update tier, score, retake timestamp, answers
```

### LM2 webhook (any source)

```
If no existing Deal and no existing Lead:
   Create Deal at stage = Received Roadmap
   Enroll Deal in FTHB LM2 - Roadmap automation (no Tier B in flight)
Else if existing Lead (and no Deal):
   Convert Lead → Deal (copy custom fields) at stage = Received Roadmap
   Enroll new Deal in FTHB LM2 - Roadmap automation
Else (existing Deal):
   If current stage is Quiz Taken:
      Advance stage to Received Roadmap
   Else (BSS Booked or later):
      No-op on stage (forward-only rule)
   If contact is currently mid-Tier-B (fthb_tier_b_completed_at is null):
      Do NOT enroll in FTHB LM2 - Roadmap yet — wait for Tier B completion trigger
   Else:
      Enroll Deal in FTHB LM2 - Roadmap automation
   In all cases: update fthb_received_lm2 = true, fthb_lm2_received_at, fthb_lm2_source
```

### Tier B completion trigger (sequential Roadmap handoff)

Fires when the final step of the `FTHB LM1 - Tier B` automation writes `fthb_tier_b_completed_at`.

```
If fthb_received_lm2 = true:
   Enroll Deal in FTHB LM2 - Roadmap automation (the 58-day long nurture)
Else:
   No-op (contact never downloaded the Roadmap; let them drop out of active nurture)
```

This is what makes the Tier B → Roadmap path sequential rather than parallel. The contact's first Roadmap nurture email (the Day 2 real-numbers email) lands 2 days after Tier B completion, not 2 days after the original quiz submission.

All later transitions (Received Roadmap → BSS Booked, BSS Booked → Pre-Approved & Searching, etc.) are **manual** — the agent moves the Deal in Pipedrive based on real-world signals (calendar bookings, lender notifications, contract events).

---

## Closed Won and Closed Lost

These are Pipedrive's built-in Deal statuses, not custom stages. Set them when a Deal terminates.

### Closed Won

Set when a Deal reaches the closing date and the transaction completes. Record:
- Closing date (Pipedrive's built-in `won_time`)
- Final purchase price (Pipedrive's built-in `value`)
- Optionally: address, lender used, brokerage commission split

### Closed Lost — suggested reason taxonomy

Pipedrive supports a configurable `lost_reason` field. For iteration 1, configure these buckets:

| Lost reason | What it captures |
|---|---|
| `financing_fell_through` | Loan denied, lender pulled out, ratio change disqualified them |
| `inspection_issues` | Couldn't agree on repair credits, major findings, walked during inspection |
| `appraisal_gap` | Appraisal came in low, parties couldn't bridge the gap |
| `buyer_changed_mind` | Personal pause, life event, lost confidence, switched to renting |
| `outbid_or_lost_negotiation` | Couldn't win on multiple offers; relevant for hot-market deals |
| `couldnt_find_suitable_home` | Searched extensively, nothing in budget / criteria fit |
| `went_cold` | Stopped responding; never reached a definitive no |
| `switched_agents` | Worked with someone else (don't take it personally; track it) |
| `other` | Free-text fallback |

Tracking lost reasons is one of the highest-ROI agent activities — after 20+ lost deals you can see which failure modes are worth investing in fixing (e.g., better lender vetting if `financing_fell_through` is over-represented).

---

## Notes on probability and rotting calibration

The numbers in the quick-reference table are starting points calibrated against industry benchmarks for buy-side residential real estate funnels with a quiz + roadmap top. They're not gospel.

After ~50 deals through the pipeline, the agent should pull actual conversion data from Pipedrive (stage-to-stage conversion rates) and re-tune. Specifically:

- If actual Quiz Taken → Won is consistently under 5% or over 15%, adjust the probability accordingly.
- If actual rotting (days a Deal spends in a stage before moving) is consistently shorter or longer than the listed rotting value, adjust.
- Pipedrive's "Funnel" report visualizes this. Run it monthly once you have ≥10 deals through.

The default rotting values err toward generous — better to under-flag than to chase deals the agent already worked.

---

## Future refinements (not iteration 1)

### Split Pre-Approved & Searching into two stages

If pipeline volume grows past ~30 active deals or the agent wants more granularity on the active-buyer phase:

| Stage | Probability | Rotting |
|---|---|---|
| **Pre-Approved** | 60% | 14 days |
| **Actively Searching** | 70% | 30 days |

Pre-Approved captures "got the letter but hasn't started touring." Actively Searching is "in the showings cycle." The split lets the agent triage between buyers who need a push to start looking (Pre-Approved, sitting too long) and buyers who are looking and need help finding a fit (Actively Searching).

Adds one manual transition per Deal in exchange for sharper triage data. Not worth it for low-volume iteration 1.

### Add a "Paused / Re-engage" stage

For deals that go quiet but aren't dead. Pipedrive's `lost_reason = went_cold` covers this fine for iteration 1, but a dedicated stage with a long rotting window (e.g., 90 days) could surface re-engagement opportunities without polluting the active pipeline. Revisit if the agent finds themselves frequently un-closing Deals to revisit them.

### Automate BSS Booked stage move from Google Calendar

Iteration 1 requires the agent to manually move a Deal to BSS Booked after seeing the calendar booking. With a Google Calendar integration into Make.com (or a Zapier bridge), this could become automated. Out of scope for now — see [`CLAUDE.md`](../CLAUDE.md) "BSS bookings are invisible to the funnel."

---

## Implementation checklist

Before turning on the Make.com scenarios:

- [ ] Pipedrive Deal pipeline configured with the six stages, in order.
- [ ] Probability set on each stage per the quick-reference table.
- [ ] Rotting set on each stage per the quick-reference table.
- [ ] `lost_reason` field configured with the bucket list above (or a subset, plus `other`).
- [ ] Make.com LM1 scenario implements the stage-setting logic for tier A/B: create at Quiz Taken or no-op stage on existing later-stage Deals.
- [ ] Make.com LM2 scenario implements the stage-setting logic: create at Received Roadmap, advance from Quiz Taken to Received Roadmap, no-op on later stages.
- [ ] Make.com Lead → Deal conversion (on LM2 webhook for Tier C Leads) creates the new Deal at Received Roadmap and copies all FTHB custom fields.
- [ ] Pipedrive funnel report set up and bookmarked for monthly review.
- [ ] Stage-move shortcuts known to the agent: how to drag a Deal between stages, how to set lost_reason, how to mark Won.
