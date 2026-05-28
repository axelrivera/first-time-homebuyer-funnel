# LM1 Readiness Filter — Email Sequences (v2)

Every email here is final copy. Drop into Pipedrive as single-email campaigns referenced by the four linear automations described below. Curly braces `{{like_this}}` are Pipedrive merge fields that resolve against the Person record Make.com has just written.

The agent's email voice = the same as the result-page voice: direct, useful, no sales-y wind-up. Short subject lines. No exclamation points unless genuinely warranted.

---

## What changed from v1 and why

Two tooling constraints make the v1 design unbuildable as written:

1. **No programmatic unenrollment.** Once a contact is enrolled in a Pipedrive automation, it can only finish on its own schedule or be stopped manually from the contact record. Make.com cannot unenroll a contact from one Pipedrive automation and enroll them in another.
2. **BSS bookings are invisible to Make.com.** Buyer Strategy Sessions are booked via Google Calendar. There is no signal back to Pipedrive when a booking happens. Any `bss_booked` field is manual-only.

v2 redesigns around those constraints with five rules:

- **Four linear Pipedrive automations, period.** Iteration 1 has exactly four automations: `FTHB LM1 - Tier A`, `FTHB LM1 - Tier B`, `FTHB LM1 - Tier C`, and `FTHB LM2 - Roadmap`. Each one is a linear sequence of single-email campaigns — no branching, no checks on field state, no mid-flow stops.
- **Routing lives in Make.com, not Pipedrive.** All decisions about which automation a contact enters happen in Make.com on the webhook. Pipedrive automations just run the emails.
- **Every sequence ends; agent reviews manually.** Every automation has a finite, defined end. After the last email, the contact is in a "no automation" state until the agent reviews them in Pipedrive and decides what's next. Default next step: add to the `FTHB Monthly Market Update`. The monthly newsletter is the only indefinite channel; everything else terminates.
- **Once enrolled, finishes.** No interruption, no manual stop for iteration 1 (the agent can still stop a campaign manually if they want, but no design assumes they will). Every automation's content must be safe to run to completion regardless of what changes about the contact mid-flight.
- **Tier A content assumes "may or may not have booked the BSS."** No "did you forget to book?" guilt emails. Every Tier A email earns its place on its own merits.

Concrete differences from v1:

| Surface                                 | v1                                                                  | v2                                                                                                                                                                |
| --------------------------------------- | ------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Pipedrive structure                     | Campaigns with workflow automations attempting mid-flight switching | **4 linear automations** orchestrating individual email campaigns; routing in Make.com                                                                            |
| Tier B automation                       | 6 emails over 21 days, pushes LM2 then BSS                          | **3 emails over 5 days**, single goal: download LM2                                                                                                               |
| Tier A automation                       | 5 emails over 14 days, BSS is the dominant CTA                      | Same cadence, **content is safe-to-finish**: value-first, soft "if you haven't booked, link is here" CTAs                                                         |
| Tier C automation                       | Bi-weekly, **indefinite** (rotating topics forever)                 | **9 emails over 8 weeks** (~2 months), weekly cadence, then ends                                                                                                  |
| Roadmap automation                      | Single Campaign with attempted cross-magnet unenrollment            | **Linear automation** (4 emails / 14 days). See `07-process-map-emails-v2.md`.                                                                                    |
| End-of-automation behavior              | Auto-roll to monthly market update                                  | **Manual review.** Each automation ends cleanly; agent decides per contact whether to add to the monthly newsletter, send a personal follow-up, or remove.        |
| Tier reassignment on retake             | Pipedrive automation unenrolls old + enrolls new                    | **Parallel run, both finish.** New tier automation enrolls (via Make.com) on retake; old one keeps running to completion. No manual stop ops needed.              |
| `bss_booked` field                      | Used as automation gate                                             | **Removed from automation**; optional manual reporting field only                                                                                                 |
| Cross-magnet handoff (LM1 Tier B → LM2) | "Automation unenrolls Tier B, enrolls LM2"                          | LM2 webhook just enrolls in the Roadmap automation. Tier B automation continues to its natural end (3 emails / 5 days — short enough that overlap is acceptable). |

Email 0 transactional carries over unchanged.

---

## Email infrastructure choices baked into this sequence

- **From name:** `{{agent_first_name}} from Orlando Homes` (or whatever the agent's working brand name is, locked in Phase 1)
- **From email:** A real address the agent monitors. Replies are responded to within 24 hours. _No no-reply addresses._
- **Pipedrive terminology** (matters for clarity): "campaign" = a single email template; "automation" = the orchestrating sequence of those campaigns.
- **Sending tools:**
  - **Pipedrive automations** for the four tier/roadmap sequences. All linear.
    - `FTHB LM1 - Tier A` (5 emails / 14 days) — primary entity: **Deal**
    - `FTHB LM1 - Tier B` (3 emails / 5 days) — primary entity: **Deal**
    - `FTHB LM1 - Tier C` (9 emails / 8 weeks) — primary entity: **Lead**
    - `FTHB LM2 - Roadmap` (4 emails / 14 days — see `07-process-map-emails-v2.md`) — primary entity: **Deal**
  - **Pipedrive automation** for **Email 0** (the LM1 transactional): a one-step automation triggered by Make.com on whichever entity (Lead or Deal) was just created or updated.
  - **Pipedrive Campaigns / Newsletter** for the **`FTHB Monthly Market Update`**: ongoing newsletter, manually populated by the agent at end-of-automation review.

### Pipedrive entity model: Person vs. Lead vs. Deal

- **Person** carries identity only: `name`, `email`, `phone`, `marketing_status`. Nothing funnel-specific.
- **All FTHB custom fields** (`fthb_lm1_tier`, scores, the per-question answers, `fthb_received_lm2`, etc.) are configured on **both Lead and Deal** entity types — identical field names and types on both — so Make.com can copy them cleanly during Lead → Deal conversion.
- **Tier C contacts are Leads.** Tier A, Tier B, and Roadmap contacts are Deals.
- The automations operate on their respective entity type (see the bullet list above). Email merge tags resolve through the Lead or Deal record into the linked Person.
- **Make.com's role:** Receive the webhook, write the Google Sheet audit row, look up or create the Pipedrive Person (identity fields only), create or update the right Lead or Deal (per the entity model + conflict rules below), write all FTHB-specific fields onto that Lead or Deal, and enroll it in the appropriate Pipedrive automation. Make.com does **not** send a single email itself — Pipedrive owns email entirely. This is the boundary; keep it clean.
- **Merge tags:** `{{like_this}}` in this doc maps to Pipedrive Campaigns merge fields. The agent will need to wire each one (`first_name`, `tier_label`, `display_score`, `fthb_result_page_link`, `book_bss_link`, `fthb_lm2_optin_link`, `fthb_retake_link`, `fthb_readiness_quiz_link`, `agent_first_name`, `agent_license_no`, `brokerage`) to the corresponding Pipedrive Person/Deal field before scheduling each campaign.
- **Manual operations the agent owns** (do not try to automate these):
  - **End-of-campaign review.** When any contact finishes their Tier A, Tier B, Tier C, or LM2 sequence, the agent opens their Pipedrive record and decides what's next. Default decision: add to `FTHB Monthly Market Update`. Other options: send a personal follow-up, leave them with no further automation, or unsubscribe them. Set up a Pipedrive view filtered to "contacts who completed a campaign in the last 7 days" so this can be batched once a week.
  - (Optional) Stopping a contact's active campaign early when a Google Calendar BSS booking comes in. Tier A content is designed to be safe to finish whether or not they booked, so this is a preference, not a requirement.
  - (Optional) Flipping `bss_booked = true` if used for reporting only.
  - Handling reply-based preference changes ("monthly", "stop") as described in the Unsubscribe section.

---

## Make.com routing logic

All routing happens in Make.com. Pipedrive automations are pure linear sequences and contain no decision logic. The Make.com scenario on each webhook decides what entity to create/update (Lead or Deal), then which automation to enroll it in. For the Deal pipeline stages Make.com sets on each webhook — and the forward-only stage-progression rule that prevents regressions on retake — see [`09-deal-pipeline-stages.md`](09-deal-pipeline-stages.md).

### On the LM1 webhook

```
1. Receive payload (includes computed tier from the client-side scoring engine)
2. Write Google Sheet audit row
3. Look up / create the Pipedrive Person by email (Person carries only name,
   email, phone, marketing_status)
4. Determine target entity from the tier:
     READY_NOW or NINETY_DAY → Deal
     FOUNDATION              → Lead
5. Resolve existing record + apply conflict rules:
     Existing Deal                    → update that Deal
     Existing Lead + target = Deal    → convert Lead to Deal (copy custom fields)
     Existing Lead + target = Lead    → update that Lead
     Existing Deal + target = Lead    → update Deal (stay a Deal, never demote)
     No existing record               → create new Lead or Deal per target
6. Write FTHB fields onto the resulting Lead or Deal:
     fthb_lm1_tier, fthb_lm1_display_score, fthb_received_lm1 = true,
     fthb_q1_credit_range … fthb_q10_lender, language, fthb_lm1_retaken_at
7. Trigger Email 0 (the LM1 transactional automation) on the Lead or Deal
8. Enroll the Lead or Deal in the corresponding tier automation:
     READY_NOW  → `FTHB LM1 - Tier A`  (Deal automation)
     NINETY_DAY → `FTHB LM1 - Tier B`  (Deal automation)
     FOUNDATION → `FTHB LM1 - Tier C`  (Lead automation)
9. Return 200
```

A contact's tier classification is mutually exclusive — exactly one of A/B/C at any given time (it's a single field value). Their automation enrollment may include parallel runs across past and current enrollments if they've retaken the quiz (see Retake handling below).

### On the LM2 webhook

```
1. Receive payload (includes source: fthb_lm1_tier_b or fthb_lm2_standalone)
2. Write Google Sheet audit row
3. Look up / create the Pipedrive Person by email
4. Target entity: always Deal (Roadmap engagement = warm enough for Deal pipeline)
5. Resolve existing record + apply conflict rules:
     Existing Deal  → update that Deal
     Existing Lead  → convert Lead to Deal (copy custom fields), then update
     No record      → create new Deal
6. Write FTHB fields onto the Deal:
     fthb_received_lm2 = true, fthb_lm2_received_at, fthb_lm2_source, language
7. Enroll the Deal in `FTHB LM2 - Roadmap` automation
8. Return 200
```

The Roadmap automation runs in parallel with whichever tier automation the contact is in (or none). No coordination between Roadmap and the tier automations — both finite, both end on their own. See `07-process-map-emails-v2.md` for the Roadmap content and the parallel-run scenarios.

Notable side-effect of the LM2 → Deal rule: any Tier C **Lead** who downloads the roadmap is automatically promoted to a **Deal**. That's intentional — they've shown deeper buyer intent. After the conversion, they're a Deal enrolled in the Roadmap automation, no longer in any Lead-side automation. If they were mid-flight in the Tier C automation, that enrollment is on the (now-converted) Lead and may be lost during conversion depending on how the agent configures the Pipedrive Lead→Deal API call. Iteration 1 accepts this — the Roadmap content is more relevant to them anyway than the back half of the Tier C educational drip.

### Optional Make.com suppressions (iteration 1.x, not required)

The agent can add suppression rules at any point without changing the Pipedrive automations:

- **Skip Roadmap enrollment for Tier C contacts** — preserves the "Tier C never sees the BSS pitch" rule strictly. Iteration 1 enrolls everyone; tighten later if Tier C contacts actually show up via the standalone path in practice.
- **Skip Tier B enrollment if the contact is already in Roadmap** — avoids the 5-day overlap where the Tier B campaign is pushing LM2 to someone who already has it. Iteration 1 lets both run; the Tier B campaign is short enough that the overlap is non-harmful.

Neither suppression is built in for iteration 1. They're noted here so future-Make.com edits know where to add them.

### Retake handling

When a contact retakes LM1, Make.com fires the LM1 webhook again with the new tier. Step 6 enrolls them in the new tier's automation. The previous tier's automation, if still running, **continues to its natural end** — Pipedrive can't be told to unenroll, and iteration 1 doesn't try.

Practical implications:

- **Upgrade (C → B or C → A):** New automation enrolls and runs. Tier C continues for up to its remaining schedule (~8 weeks). Both end on their own. Agent reviews each at end.
- **Upgrade (B → A):** Tier A enrolls and runs. Tier B continues (max 5 more days). Brief overlap, self-resolves.
- **Downgrade (A → C or B → C):** Tier C enrolls and runs. The previously-running tier finishes. Tier C is non-pitching educational content; if the agent thinks it's redundant for someone who was already at a higher tier, they can manually stop the Tier C enrollment in Pipedrive. Default — let it run — is fine.

### Custom fields Make.com maintains

- `fthb_received_lm2` (boolean) — set true on LM2 webhook. Used for reporting / agent visibility, not for any automation gate.
- All the field writes listed in the scenarios above (`fthb_lm1_tier`, `fthb_lm1_display_score`, etc.) are for record-keeping on the Pipedrive Person; none of them drive routing logic inside Pipedrive automations.

---

## Email 0 (all tiers): Transactional delivery

**Trigger:** Immediately on form submit (within 60 seconds), via Make.com calling a one-step Pipedrive automation.
**Goal:** Confirm receipt, deliver the result link, set the tone.
**Surface:** A standalone one-step Pipedrive automation that sends a single campaign email. Not part of the Tier A/B/C automations — fires for every LM1 submission regardless of tier.

**Subject:** Your Orlando readiness score: {{tier_label}}
**Preview text:** Score, timeline, and the 2 mistakes to avoid. All inside.

```
Hey *|FIRST_NAME|*,

You came back as {{tier_label}}, {{display_score}}/100.

Here's your tier guide as a PDF. It walks through your tier
explanation, the 2 mistakes buyers in your tier most often
make, and a 1-page Orlando market snapshot:

    {{fthb_tier_pdf_link}}

Your next step is on the last page.

Two things to know:

1. I read every reply. If anything in the guide raises a
   question (about your score, the market data, the next step,
   anything), just hit reply.

2. I'm not going to call you, text you, or pass your info to
   anyone. Not now, not ever. If we end up talking, it'll be
   because you booked something on your own.

Talk soon,
{{agent_first_name}}
{{agent_license_no}} | {{brokerage}}

P.S. Save the PDF or print it. The 2-mistakes section gets a
lot more useful the day before you actually talk to a lender,
so come back to it then.
```

---

## Tier A: "Ready Now" sequence (5 emails)

**Automation name:** `FTHB LM1 - Tier A`
**Primary entity:** Deal. Make.com creates (or promotes a Lead to) a Deal on LM1 webhook when tier = READY_NOW; the automation enrolls and operates on that Deal.
**Surface in Pipedrive:** a linear automation orchestrating 5 single-email campaigns. No branching, no checks.
**Cadence:** Email A1 ~10 minutes after the transactional. Then days 2, 5, 9, 14.
**Goal:** Be useful to a Tier A buyer whether they book a BSS, already booked one, or never book. The BSS link appears in every email as a soft CTA at the end; the body of every email earns its keep regardless of booking state.
**Design contract:** No "did you forget to book?" framing. No countdown urgency. Each email is a piece of pre-call prep (if they booked) or standalone value (if they didn't).

### Email A1 — Day 0 (sent ~10 minutes after transactional)

**Subject:** One thing about your "Ready Now" score

```
Hey *|FIRST_NAME|*,

Quick follow-up on your scorecard result.

"Ready Now" doesn't just mean you can technically qualify for a
mortgage. It means the gap between you and keys is short enough
that timing matters in a way it doesn't for other tiers.

Here's the thing nobody tells first-time buyers in your spot:

The two months between pre-approval and closing are the months
where deals fall apart the most. Not because of the market,
but because of avoidable mistakes the buyer makes (new credit,
large deposits, job changes). The 30 days before closing are
the most fragile.

Three rules that cover 90% of it:

  1. Do not open any new credit line, including a store card
     "to get 10% off the appliances."
  2. Do not make any deposit over $1,000 that can't be cleanly
     traced to your paycheck or a documented gift.
  3. Do not change jobs, even laterally, until after closing.

Save this email. The week before closing, reread it.

If you want to talk through the specific window between where
you are now and keys in hand, the 30-minute Buyer Strategy
Session is the place. Free, video call.

    {{book_bss_link}}

If we've already got a call on the calendar, bring any questions
about your specific timeline and we'll dig in.

- {{agent_first_name}}
```

### Email A2 — Day 2

**Subject:** Seminole County vs. Orange County: a 90-second read

```
Hey *|FIRST_NAME|*,

Most "Ready Now" buyers in Orlando default to one of two
paths:

  Path 1: Buy on the Orange side (Apopka, Maitland) — Apopka
  for new-construction value north of Downtown Orlando, Maitland
  for walkability and proximity to Park Avenue at Winter Park
  prices minus $100K, but Orange school zoning is more uneven
  block to block

  Path 2: Push north on the Seminole side (Altamonte Springs,
  Longwood, Lake Mary, Winter Springs, Oviedo, Sanford) —
  tighter inventory in the at-anchor cities, longer Downtown
  Orlando commute from Sanford, but the strongest school-zoning
  resale floor in the metro

There's a third path most people miss: the **Seminole/Orange
border zone** — Casselberry and southern Altamonte Springs
(Seminole) sitting right against Maitland and northern Winter
Park (Orange). Specific blocks let you live within a 10-minute
drive of both county school systems and choose your trade-off
street by street, with meaningful price-per-square-foot
differences between the two sides.

If your budget can stretch to $575-600K, there's a fourth path
worth a look: Lake Nona in the southeast. Newer construction,
master-planned, totally different feel from anything on the
north end. Not for everyone, but it should be on the table
before you rule it out.

If you've got a call on the calendar already, bring your top 3
target areas (or just "I have no idea where to start") and we'll
walk the map together. If you haven't booked yet and want to,
the link is here:

    {{book_bss_link}}

- {{agent_first_name}}
```

### Email A3 — Day 5

**Subject:** The pre-approval question I always ask first

```
Hey *|FIRST_NAME|*,

When someone in the "Ready Now" tier reaches out, the very first
question I ask is:

"Did your lender pre-approve you, or did they pre-qualify you?"

These sound the same. They aren't.

  - Pre-qualification = you told them your numbers and they did
    quick math. No documents pulled. No underwriting. Useless on
    a competitive offer.

  - Pre-approval = they pulled credit, verified income, verified
    assets, and ran it past an underwriter.

If you have a letter, look at it now. If it doesn't say
"pre-approved" (and most don't), that's the first thing to fix
before you make an offer on anything.

If you're not sure, you can send me a redacted copy of your
letter and I'll tell you in 5 minutes — whether we've already
got a call on the calendar or not. If we have, send it ahead
and I'll have feedback ready when we meet.

Reply with the letter, or → {{book_bss_link}}

- {{agent_first_name}}
```

### Email A4 — Day 9

**Subject:** The one contingency that does the heavy lifting

```
Hey *|FIRST_NAME|*,

Quick one.

When I closed on my own first home in Altamonte Springs in
January 2024, the part of the contract that gave me the most
peace of mind was not the financing contingency or the
appraisal. It was the inspection period.

In a Florida purchase contract, the inspection period is the
one contingency that lets you walk away for almost any reason
and keep your deposit.

Not just "the inspector found something." It is your window
to decide whether the home, the neighbors, the HOA documents
you finally got around to reading, the commute you tested at
8am, and the noise at night are actually right for you.

The other contingencies are narrower. Appraisal protects you
if the home appraises below the contract price. Financing
protects you if your loan falls through. Both useful, both
specific.

The inspection period is the broad one. Use it that way.

How to actually use your contingencies is one of the higher-
leverage things we cover on the strategy session. If we've
already got one booked, bring offer-mechanics questions. If
not, the link is here:

    {{book_bss_link}}

- {{agent_first_name}}
```

### Email A5 — Day 14

**Subject:** Wrapping the email series

```
Hey *|FIRST_NAME|*,

This is the last email in the Ready Now series.

Two scenarios:

  - We've already talked, or we have a call on the calendar.
    Great. You've got specific next steps from that conversation
    (or will). After this email, I'm moving you to the monthly
    market update — one email a month, Orlando-specific.

  - We haven't talked yet. Also fine. You have the result page,
    you have the action items from the last few emails, and you
    can absolutely do this on your own. You're also moving to
    the monthly market update. If you ever want to talk, the
    link is below — no awkwardness, no hard sell.

    {{book_bss_link}}

- {{agent_first_name}}

P.S. If a friend in Orlando is at the same stage you are, the
scorecard link is open: {{fthb_readiness_quiz_link}}. No referral
fee, no tracking. I'd just rather more first-time buyers know
where they actually stand before they start shopping.
```

After A5, the campaign ends. The agent reviews the contact in Pipedrive and decides next steps — typically adding them to the `FTHB Monthly Market Update`, but the decision is per-contact.

---

## Tier B: "90-Day Sprint" sequence (3 emails)

**Automation name:** `FTHB LM1 - Tier B`
**Primary entity:** Deal. Make.com creates (or promotes a Lead to) a Deal on LM1 webhook when tier = NINETY_DAY; the automation enrolls and operates on that Deal.
**Surface in Pipedrive:** a linear automation orchestrating 3 single-email campaigns. No branching, no checks.
**Cadence:** Email B1 ~10 minutes after the transactional. Then days 2 and 5.
**Goal:** Get the contact to download LM2 (the 10-Step First Home Roadmap). That's the whole job. No BSS pitching in this automation — the BSS lives downstream in the `FTHB LM2 - Roadmap` automation (see `07-process-map-emails-v2.md`).
**Design contract:** Short enough that "letting it finish" is harmless. If the contact downloads LM2 partway through, Make.com enrolls them in the Roadmap automation; this Tier B automation continues to its natural end either way — its remaining 0–2 emails are non-harmful since they share the same LM2 link.

### Email B1 — Day 0 (sent ~10 minutes after transactional)

**Subject:** Your 90-day game plan: the 10 steps

```
Hey *|FIRST_NAME|*,

You came back as 90-Day Sprint, which means the next 90 days
are the whole game. Pre-approval, target areas, savings, and
offer prep all happen in a specific order — miss the order and
the 90 days becomes 180.

On the result page, your "what's next" was to grab my roadmap.
If you didn't snag it there, here's the direct link:

    {{fthb_lm2_optin_link}}

It's called **The 10-Step First Home Roadmap** — exactly what
happens between "I think I'm ready" and keys in your hand in
Orlando. Every step shows what you do, what your agent does,
and how long it takes.

Three things I built into it you won't find in a generic buyer
guide:

  - The 3 places first-time buyers in Orlando lose money
    (named, specific, avoidable)
  - HOA disclosure timelines and Florida rainy-season inspection
    timing (local stuff)
  - The one number on your credit report that affects your rate
    more than your score does

Grab it. Spend 20 minutes with it. Then you'll know exactly
which step you're on.

- {{agent_first_name}}
```

### Email B2 — Day 2

**Subject:** The order matters more than most people think

```
Hey *|FIRST_NAME|*,

If you haven't grabbed the roadmap yet, here's the single
biggest mistake I see Sprint-tier buyers make:

They start going to open houses before they have a pre-approval.

I get it. Open houses feel productive. They aren't, not in
this tier. Here's what actually happens:

  1. You walk through 4 houses on a Sunday.
  2. You fall in love with one.
  3. You go to a lender on Monday.
  4. The lender tells you that house was $40K above what you'd
     comfortably qualify for.
  5. You spend the next 3 weeks comparing every house you see
     to the one you can't have.

The fix is to do those steps in the other order. Pre-approval
first. Then walk through houses in your actual range. You will
see houses you like, I promise.

That's Step 3 in the roadmap. Steps 1-2 (credit prep and
realistic budget) are what make Step 3 produce a number you
can actually trust.

    {{fthb_lm2_optin_link}}

- {{agent_first_name}}
```

### Email B3 — Day 5

**Subject:** Last note on the roadmap

```
Hey *|FIRST_NAME|*,

Closing out this short series.

The roadmap link is below one more time. If it's not for you,
no problem — you're moving to the monthly market update either
way. One email a month, Orlando-specific, no pitch.

    {{fthb_lm2_optin_link}}

If you grab it, the first follow-up email will hit your inbox
within a few minutes and we'll start working through the 9
steps together.

- {{agent_first_name}}

P.S. The number-one reason I built the roadmap as a standalone
document instead of a call: most 90-day buyers aren't sure
they're really 90 days out. The roadmap lets you check that
honestly before anyone tries to sell you a strategy session.
```

After B3, the automation ends. The agent reviews the contact in Pipedrive and decides next steps. If `fthb_received_lm2 = true` by the end of B3, the contact is already enrolled in the `FTHB LM2 - Roadmap` automation (see `07-process-map-emails-v2.md`), so the default review action is "no further enrollment needed — let the Roadmap automation play out, then review again at its end." If LM2 wasn't downloaded, the typical action is to add the contact to the `FTHB Monthly Market Update`. The Tier B automation continues to its natural end regardless — its remaining 0–2 emails are non-harmful since they share the same LM2 link.

---

## Tier C: "Foundation Phase" sequence (9 emails over 8 weeks)

**Automation name:** `FTHB LM1 - Tier C`
**Primary entity:** Lead. Make.com creates a Lead on LM1 webhook when tier = FOUNDATION; the automation enrolls and operates on that Lead. If the Lead later downloads LM2 or retakes to a higher tier, Make.com converts it to a Deal (per the conflict rules) and this Tier C automation enrollment is left behind on the converted record.
**Surface in Pipedrive:** a linear automation orchestrating 9 single-email campaigns. No branching, no checks.
**Cadence:** Weekly. C0 (welcome) immediately, then 8 educational emails every 7 days. Total: **9 emails over 56 days (~2 months)**. After C8 (the wrap-up), the automation ends.
**Goal:** Maximum-value compressed education for a Foundation-Phase buyer. Do not pitch the BSS. The bar for every email is _"is this concretely useful to someone 6–12 months out from buying in Orlando — and is it specific enough that they could act on it this week?"_
**Design contract:** Finite. The agent reviews each contact at C8's end and decides what's next — typically adding to the `FTHB Monthly Market Update` to limit content management overhead.

### Email C0 — Day 0 (welcome)

**Subject:** What "Foundation Phase" actually means

```
Hey *|FIRST_NAME|*,

Foundation Phase doesn't mean "no." It means "not yet, and here
is what to do with the time."

You're on my Foundation list now. Here's what to expect:

  - 8 short emails over the next 2 months, one a week
  - 100% useful, 0% sales pitch
  - Topics: credit, savings, down payment math, DTI, FHA vs.
    conventional, Florida assistance money, lease timing,
    and how to know when you're ready

When you cross into the 90-Day Sprint tier (which usually means
your credit, savings, and timeline all line up), I'll let you
know. You can retake the scorecard anytime to check:

    {{fthb_retake_link}}

First real email comes in next week.

- {{agent_first_name}}
```

### Email C1 — Day 7: Credit

**Subject:** The one place to actually pull your credit (and why)

```
*|FIRST_NAME|*,

There are three credit-related things every Foundation Phase
buyer should do this month. They cost nothing and they take
under an hour.

  1. Go to annualcreditreport.com, not Credit Karma, not your
     bank app, not "free credit score" websites. This is the
     government-mandated site where you can get your real
     report from all three bureaus, free, once a year.

  2. Look at three numbers: your score, your "credit
     utilization" (the % of your credit limits you're using),
     and any past-due accounts.

  3. If utilization is above 30%, that's the fastest place to
     move your score in 60–90 days. Pay revolving balances
     down below 30% of the card limit. Don't close the cards
     after; that hurts your score by lowering your total
     available credit.

That's it for this week.

- {{agent_first_name}}
```

### Email C2 — Day 14: Savings

**Subject:** Where to actually park your home fund

```
*|FIRST_NAME|*,

If you're saving for a down payment, the worst place that money
can be is in a regular checking account. Two reasons: it earns
nothing, and it gets spent.

The right place is a high-yield savings account at an online
bank. They pay meaningfully more than a checking account, for
the same money in the same FDIC-insured way. One form to fill
out.

Rates move with the Fed, so shop around when you open one and
recheck once a year. A few I won't endorse but you can compare
yourself: Marcus, Ally, Wealthfront, Apple Savings. Look at the
APY and the FDIC insurance limit; that's it.

Bonus move: name the account "Orlando Home Fund." Sounds dumb,
but it works. It's measurably harder to pull money out of an account
with a goal name on it.

- {{agent_first_name}}
```

### Email C3 — Day 21: Down payment myths

**Subject:** Drop the 20% number from your head

```
*|FIRST_NAME|*,

If "I need 20% down" is the number sitting in your head, swap
it for the real one before you do anything else.

20% is a 1970s-era benchmark. It does one thing today: it lets
you skip private mortgage insurance (PMI). That's it. It is
not required to buy a house. Most first-time buyers in Orlando
do not put 20% down.

Real minimums, by loan type:

  - FHA: 3.5% down. Most first-time buyers I work with use
    this.
  - Conventional 97 / HomeReady / Home Possible: 3% down.
    For credit scores above ~680.
  - VA (military / veteran): 0% down.
  - USDA (some areas just outside the Orlando metro qualify):
    0% down.

On a $325K home (a realistic below-anchor FTHB price in
Seminole County — think Sanford, Longwood, or a smaller place
in Winter Springs), the math breaks down like this:

  - 20% down  = $65,000
  - 3.5% down (FHA)            = $11,375
  - 3% down  (Conventional 97) = $9,750

The $50,000+ gap between 20% and 3.5% is often the gap between
"I'll be ready in 8 years" and "I'll be ready in 18 months."

For most buyers, the right move is not to save 20%. It is to
put down the minimum, accept PMI for the few years it takes
to build to 20% equity, and cancel PMI then. Two weeks from
now I'll send you the FHA-vs-Conventional decision rules,
including which kinds of PMI you can cancel and which you
can't.

- {{agent_first_name}}
```

### Email C4 — Day 28: Lease renewal timing

**Subject:** Before you sign that 12-month renewal

```
*|FIRST_NAME|*,

Most Foundation Phase buyers I talk to are renters. Sooner or
later, your landlord asks: "Do you want to renew for 12
months?" Here's how to decide.

Sign the 12-month renewal if any of these are true:

  - Your credit score is below where you want it for a
    pre-approval, and you're working on it for the next 6+
    months
  - Your savings are less than 80% of your target number
  - You have never spoken to a lender (so you don't actually
    know your real timeline yet)

In all three cases, a 12-month commitment costs you nothing
you weren't going to do anyway. And month-to-month rent in
Orlando is usually $200–$400/month more than a year-lease
rate, which is real money to pay for flexibility you don't
need yet.

Negotiate a 6- or 9-month renewal if:

  - You're 4–9 months from a real pre-approval timeline
  - Your landlord is willing (some are, especially when the
    rental market is soft — Orlando 2026 is mixed; ask)

Go month-to-month if:

  - You're within 60 days of making an offer
  - You have a real pre-approval letter in hand
  - You're prepared to absorb the premium for 1–2 months

The trap to avoid: signing a 12-month renewal one month
before you would have been ready to buy, because nobody asked
the question 90 days earlier.

Already mid-renewal? Most Florida leases have a buyout clause,
usually 2 months' rent. Annoying but not blocking.

- {{agent_first_name}}
```

### Email C5 — Day 35: DTI

**Subject:** The number lenders actually care about

```
*|FIRST_NAME|*,

Most first-time buyers obsess over their credit score and
ignore their debt-to-income ratio (DTI). Lenders do the
opposite. Score gets you in the door; DTI gets you the loan
amount.

DTI is a percentage. The math:

  DTI = (all your monthly debt payments) ÷ (gross monthly income)

"All monthly debt payments" includes minimum credit card
payments, car loans, student loans, child support, and — most
importantly — the proposed mortgage payment (P+I + property
tax + insurance + HOA + PMI if any).

Lenders draw two lines:

  - Front-end DTI (housing payment alone, as % of income):
    most prefer under 28%. FHA accepts up to 31%.
  - Back-end DTI (housing payment + all other debt):
    most prefer under 43%. FHA accepts up to 50% in many
    cases.

The number that matters most in Orlando 2026 is the back-end.
Most first-time buyers I see qualify for less than they
expected because their car payment + student loans push their
back-end DTI past 43%.

A 15-minute exercise this week:

  1. List every monthly debt payment (minimums, not balances)
     on one page.
  2. Add a placeholder mortgage payment of $2,500 (a rough
     2026 estimate for a $300K FHA loan in Seminole County —
     P+I + taxes + insurance + MIP).
  3. Divide the total by your gross monthly income.

If that number is above 43%, your highest-leverage move is
not saving more — it's paying off the debt with the smallest
balance and highest minimum. One paid-off car loan can shift
your DTI by 5–8 points, which can be the difference between
qualifying for a starter home in Sanford and not.

- {{agent_first_name}}
```

### Email C6 — Day 42: FHA vs. Conventional

**Subject:** FHA or Conventional? Two rules of thumb.

```
*|FIRST_NAME|*,

The biggest loan-choice decision most first-time buyers in
Orlando face: FHA or Conventional? Both work. The trade-offs
are specific.

Choose FHA when:

  - Your credit score is 580–679
  - Your DTI is in the 43–50% range (FHA tolerates higher)
  - You have 3.5% down but not 5%
  - You have any past-due history or recent credit hiccups

Choose Conventional (specifically Conventional 97, HomeReady,
or Home Possible) when:

  - Your credit score is 680+
  - Your DTI is under 43%
  - You have 3–5% down
  - Your goal is to drop PMI within 2–3 years

The PMI difference is the big one in Florida.

On a new FHA loan, the mortgage insurance is **permanent for
the life of the loan**. The only way out is refinancing into
a Conventional loan later.

On a Conventional loan, PMI is **cancelable** at 20% equity.
Orlando home prices have appreciated steadily for the last
several years. If you put 5% down on a Conventional 97 loan
in 2026 and prices appreciate even modestly for 3 years, you
may be at or near 20% equity and able to drop PMI — saving
$150–$280/month from that point on.

My rule of thumb:

  - 580–679 credit + 3.5% down → FHA, period.
  - 680–719 credit + 3% down  → Conventional 97, almost
    always.
  - 720+ credit + 5%+ down     → Conventional, every time.

Edge cases (recent self-employment, gift funds, manual
underwrites, non-traditional credit) shift the calculus.
Those are conversations with a lender, not emails.

- {{agent_first_name}}
```

### Email C7 — Day 49: Florida down-payment assistance

**Subject:** The Florida money most buyers never claim

```
*|FIRST_NAME|*,

The single most under-claimed money on the table for
first-time buyers in Florida is **Florida Housing's**
down-payment-assistance programs. Most Foundation Phase
buyers I talk to have never heard of them.

The headline programs (current as of 2026 — verify with a
Florida-Housing-approved lender, the state updates these):

1. **Florida Hometown Heroes Loan Program**

  - Up to $35,000 in down-payment + closing-cost assistance
  - Targeted at workers in eligible Florida occupations:
    teachers, healthcare workers, law enforcement,
    firefighters, military, and a broad list of others
  - Used alongside a primary FHA, VA, USDA, or Conventional
    loan
  - Must be a first-time buyer or not have owned a home in
    the last 3 years

2. **Florida Assist (FL Assist)**

  - Up to $10,000 in down-payment + closing-cost assistance
  - Statewide, for first-time buyers
  - Used with a Florida Housing first mortgage

3. **HFA Preferred (Florida Housing 3%)**

  - 3% of the loan amount toward down payment and closing
    costs
  - Conventional loan, often with reduced PMI

The catches:

  - Must use a Florida-Housing-approved lender. Not every
    lender qualifies, and the list changes.
  - Must complete an approved homebuyer education course
    (online, 4–8 hours, ~$75).
  - These are forgivable or repayable second mortgages, not
    grants. Read the terms.
  - Funding runs out periodically and reopens when the state
    refunds the program. If a lender says "Hometown Heroes is
    paused right now," that's normal — check back in 30–60
    days.

Your move this week: visit FloridaHousing.org, scan the
program list, see whether your occupation qualifies for
Hometown Heroes, and bookmark the approved-lender finder.
When you eventually have a real pre-approval conversation,
ask the lender directly: "Are you Florida-Housing-approved,
and do you know which program fits my situation?" If they
hesitate or don't know, find a different lender.

- {{agent_first_name}}
```

### Email C8 — Day 56: Wrap-up

**Subject:** Closing out the Foundation series

```
*|FIRST_NAME|*,

This is the last email in the Foundation series.

Eight weeks ago you took the scorecard and landed in
Foundation Phase, which meant the work for you was credit,
savings, lender prep, and finding what Florida-specific money
was on the table.

You now have the playbook. The next move is yours.

Three things worth doing this week:

  1. Retake the scorecard. Two months of focused work moves
     most first-time buyers up a tier. Find out where you
     actually stand now:

       {{fthb_retake_link}}

  2. If you've made enough progress to be inside 90 days,
     reply to this email and tell me. I'll personally walk
     you through what comes next.

  3. If you're not there yet, that's totally fine. Foundation
     Phase is a real tier, not a euphemism for "not good
     enough." Most great first-time-buyer stories start here.

After this email, the auto-series ends. I'll review your
contact in the next week and figure out what makes sense from
here — most likely my monthly Orlando market update (one
email a month, no pitch), but I'll think about it per person.

If you'd rather skip whatever I have in mind, just reply
"unsubscribe" and I'll take you off entirely.

Either way: you have my email. Reply when something changes.

- {{agent_first_name}}
```

After C8, the campaign ends. The agent opens the contact in Pipedrive and decides next steps — default is "add to `FTHB Monthly Market Update`" to limit content-management overhead.

---

## Monthly newsletter — `FTHB Monthly Market Update`

**Surface in Pipedrive:** an ongoing newsletter / recurring campaign (not an automation in the Tier sense). One email per month.
**Cadence:** Once monthly, indefinite. **This is the only indefinite channel in the funnel** — every other automation terminates.
**Enrollment:** Manual. The agent adds contacts at the end of their primary automation (Tier A, Tier B, Tier C, or Roadmap) as part of the end-of-automation review. No automated handoff enrolls anyone here.
**Goal:** Stay relevant for contacts who finished a primary sequence without converting. Orlando-specific market data, no pitch, one CTA at the bottom ("if anything changed for you, reply or rerun the scorecard: {{fthb_retake_link}}").

Topics rotate through: median price movement across the Orlando metro (anchored in the north with Winter Garden and Lake Nona as outliers — the same 10 areas the market snapshot covers), inventory by price band in the FTHB range, interest-rate context, one neighborhood deep-dive per month (rotating among Altamonte Springs, Apopka, Sanford, Winter Springs, Longwood, Lake Mary, Oviedo, Maitland, Winter Garden, Lake Nona).

Format and copy for monthly emails are not locked in this doc — they'll be drafted on a rolling basis during Phase 4 against real submission data.

---

## Unsubscribe / preference handling

Every email carries the standard Pipedrive unsubscribe footer (required for CAN-SPAM compliance and provided by the Pipedrive template). Additionally, Tier C emails carry this line near the bottom:

> _Don't want these every week? Reply "monthly" and I'll switch you to once a month. Reply "stop" and I'll take you off entirely._

The standard unsubscribe link removes them from all Pipedrive automations + the newsletter. The "monthly" reply is handled manually by the agent (stop the current automation; add directly to the monthly newsletter). The "stop" reply triggers the standard unsubscribe in Pipedrive. Both work without code on the static site.

---

## Implementation checklist

Before turning anything on:

- [ ] **Pipedrive entity setup:** custom fields configured on **both Lead and Deal** entity types (same field name + type on both, so Make.com can copy during Lead→Deal conversion):
  - `fthb_lm1_tier` (enum: READY_NOW / NINETY_DAY / FOUNDATION)
  - `fthb_lm1_display_score` (number)
  - `fthb_lm1_retaken_at` (datetime)
  - `fthb_received_lm1` (boolean)
  - `fthb_received_lm2` (boolean)
  - `fthb_lm2_received_at` (datetime)
  - `fthb_lm2_source` (enum: fthb_lm1_tier_b / fthb_lm2_standalone)
  - `fthb_q1_credit_range` … `fthb_q10_lender` (per-question enums)
  - `language` (enum: en / es)
- [ ] Person record holds only `name`, `email`, `phone`, `marketing_status` — no funnel fields on Person.
- [ ] Merge tags wired to resolve from the Lead or Deal (and the linked Person for `first_name`): `first_name`, `tier_label`, `display_score`, `fthb_result_page_link`, `book_bss_link`, `fthb_lm2_optin_link`, `fthb_retake_link`, `fthb_readiness_quiz_link`, `agent_first_name`, `agent_license_no`, `brokerage`.
- [ ] Email templates (single-email Pipedrive campaigns) built:
  - 5 for Tier A (A1–A5)
  - 3 for Tier B (B1–B3)
  - 9 for Tier C (C0–C8)
  - 1 for Email 0 (LM1 transactional)
- [ ] **Five Pipedrive automations** built and linear, each on the correct primary entity:
  - `Email 0 - LM1 Transactional` — one-step automation; fires on the Lead or Deal Make.com just touched.
  - `FTHB LM1 - Tier A` — 5 campaigns in sequence (A1 → A5). Entity: **Deal**.
  - `FTHB LM1 - Tier B` — 3 campaigns in sequence (B1 → B3). Entity: **Deal**.
  - `FTHB LM1 - Tier C` — 9 campaigns in sequence (C0 → C8). Entity: **Lead**.
  - `FTHB LM2 - Roadmap` — 4 campaigns in sequence (Email 0 + N1 + N2 + N3). Entity: **Deal**. Content in `07-process-map-emails-v2.md`.
- [ ] Monthly newsletter (`FTHB Monthly Market Update`) exists as an ongoing recurring campaign in Pipedrive. First issue drafted closer to Phase 4. Enrollment is manual only.
- [ ] **Make.com LM1 scenario:** writes/looks up Person, applies conflict-resolution rules to create or update the right Lead or Deal (per the "Make.com routing logic" section above), writes FTHB fields, triggers Email 0 automation, enrolls in the tier-appropriate automation, returns 200.
- [ ] **Make.com LM2 scenario:** writes/looks up Person, applies conflict-resolution rules (always targeting Deal — converts any existing Lead), writes FTHB fields, enrolls in the Roadmap automation, returns 200.
- [ ] **Make.com Lead→Deal conversion logic:** dedicated step that copies all FTHB custom fields from the Lead to the new Deal before marking the Lead converted. Without this, custom field history is lost.
- [ ] Pipedrive views set up: "Deals whose automation completed in the last 7 days" + "Leads whose automation completed in the last 7 days" so the agent can batch end-of-automation review weekly.
- [ ] Manual ops documented for the agent: end-of-automation review (typical action: add to monthly newsletter), reply-based preference changes ("monthly", "stop").
- [ ] Monthly market update campaign exists as a placeholder; first issue drafted closer to Phase 4. Enrollment is manual only (no automation enrolls anyone here).
- [ ] Pipedrive view set up: "Contacts whose campaign ended in the last 7 days" so the agent can batch end-of-campaign review weekly.
- [ ] Manual ops documented for the agent: end-of-campaign review (typical action: add to monthly newsletter), reply-based preference changes ("monthly", "stop").
