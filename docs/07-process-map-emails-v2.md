# LM2 Process Map — Email Sequence (v2)

Same email rules as LM1: real from-address, no no-reply, replies answered within 24 hours. In Pipedrive terminology, "campaign" = a single email template; "automation" = the orchestrating sequence. This doc describes the **`FTHB LM2 - Roadmap` automation**, which is a single linear sequence of 4 emails (1 transactional + 3 nurture).

The Roadmap automation runs in parallel with whichever Tier automation a contact is in (Tier A, B, or C). Both finite, both end on their own, no interaction between them.

---

## What changed from v1 and how this iteration differs from earlier v2 drafts

v1 designed the Roadmap as a single Pipedrive Campaign with one set of emails plus an automated handoff that was supposed to unenroll a Tier B contact from their existing campaign and enroll them in the Roadmap one. That handoff isn't buildable in Pipedrive.

Earlier v2 drafts overcorrected by building a branching Pipedrive Workflow Automation with three content variants per step (Tier B / Tier C / Standalone) plus a STOP branch for Tier A. That was over-engineered for iteration 1 — branching automations are harder to maintain, harder to visualize, and unnecessary when routing logic can live in Make.com instead.

The current iteration is the simplest version that works:

- **One linear Pipedrive automation** (`FTHB LM2 - Roadmap`) with 4 emails total. No branching, no variants, no STOP, no checks on contact field state.
- **All routing decisions live in Make.com.** Make.com decides who enters this automation (and who doesn't) based on the LM2 webhook context.
- **Same emails for everyone enrolled.** The content level-sets all readers; the final email's BSS CTA is soft enough to work across Tier A / B / no-quiz audiences.
- **Once enrolled, finishes.** No way to stop mid-flight; not designed to be stopped.

---

## Infrastructure

- **Surface in Pipedrive:** one automation named `FTHB LM2 - Roadmap`, which orchestrates 4 individual email campaigns (1 transactional + 3 nurture) in sequence.
- **Primary entity:** **Deal.** Downloading the roadmap is treated as warm-enough engagement to land in the Deal pipeline, not the Lead pool. See `04-readiness-filter-emails-v2.md` "Pipedrive entity model" for the Person / Lead / Deal split and `04-readiness-filter-emails-v2.md` "Make.com routing logic" for the conflict-resolution rules. The short version: Make.com creates a new Deal if no record exists, updates the Deal if one exists, and **converts a Lead to a Deal** if the contact was previously a Tier C Lead.
- **From name, from email, voice:** identical to LM1 (see [04-readiness-filter-emails-v2.md](04-readiness-filter-emails-v2.md)).
- **Merge tags new to LM2** (in addition to the LM1 set): `fthb_roadmap_view_link`, `fthb_roadmap_pdf_link`.

## Trigger and routing (Make.com side)

Make.com owns the routing decision on the LM2 webhook. The webhook fires for both `source = "fthb_lm1_tier_b"` and `source = "fthb_lm2_standalone"`. For the Deal pipeline stages Make.com sets — including the LM2 → "Received Roadmap" stage transition and the forward-only progression rule — see [`09-deal-pipeline-stages.md`](09-deal-pipeline-stages.md).

Make.com's scenario on the LM2 webhook:

1. Receive the payload.
2. Write the Google Sheet audit row.
3. Look up / create the Pipedrive Person by email (Person carries only `name`, `email`, `phone`, `marketing_status`).
4. Target entity: **always Deal** for the Roadmap path. Apply conflict-resolution rules:
   - Existing Deal → update that Deal.
   - Existing Lead → **convert the Lead to a Deal** (copy all FTHB custom fields).
   - No existing record → create a new Deal.
5. Write FTHB fields onto the resulting Deal: `fthb_received_lm2 = true`, `fthb_lm2_received_at = now`, `fthb_lm2_source = <source>`, language toggle if changed.
6. Enroll the Deal in the `FTHB LM2 - Roadmap` automation.
7. Return `200`.

The agent can add suppression logic to step 6 at any time (e.g., "if the Deal was just converted from a Tier C Lead and the current `fthb_lm1_tier = FOUNDATION`, don't enroll" — to keep those contacts from receiving the soft BSS pitch in the final email). Iteration 1 enrolls everyone who hits the webhook regardless of prior tier; that's the simplest starting point.

Notable side effect: a contact who was a **Tier C Lead** before this webhook becomes a **Deal** here. Their previous enrollment in the `FTHB LM1 - Tier C` automation (which operates on Leads) is left behind on the converted Lead record. Whether it continues to send the remaining Tier C emails depends on how the agent configures the Pipedrive Lead→Deal API call. Iteration 1 accepts losing the remaining Tier C drip — the Roadmap content is more relevant to a contact who actively engaged at a deeper level.

---

## The four emails

The Roadmap automation sends these four emails in order. Day 0 immediately, then Day 3, Day 7, Day 14. Total span: 14 days.

### Email 0 — Day 0: Transactional delivery

**Subject:** Your 9-Step Orlando Home Roadmap (link + PDF inside)
**Preview text:** The full roadmap, plus where to start reading depending on where you are.

```
Hey {{first_name}},

Here's the 9-Step First Home Roadmap as promised.

  → Read it on the web: {{fthb_roadmap_view_link}}
  → Download the PDF (for offline reading or printing):
    {{fthb_roadmap_pdf_link}}

If you took the Readiness Scorecard and landed in the 90-Day
Sprint tier, **start at Step 4** (lender pre-approval). That's
the bottleneck for almost everyone in that tier.

If you came in from the roadmap landing page directly (no
scorecard yet), start at the visual roadmap and find the step
that looks most like where you are. The scorecard is a 7-minute
way to get a more specific starting point:

    {{fthb_readiness_quiz_link}}

Two notes before you dive in:

1. The roadmap is dense. That's intentional. Skim it once
   end-to-end first, then come back to the step you're in.

2. If a step raises a question, hit reply. I read every email
   and I usually respond within a day.

Talk soon,
{{agent_first_name}}

P.S. The roadmap link is a shareable URL. If you know someone
in Orlando who's a few steps behind you on this, send it. I'd
rather they get the map than figure it out the expensive way.
```

### Email N1 — Day 3

**Subject:** Which step did you start on?

```
Hey {{first_name}},

Quick check-in.

You got the roadmap a few days ago. Most people who open it
start at one of two places:

  1. The visual diagram at the top: they want the bird's-eye
     view first.
  2. The Pre-Approval Cheat Sheet at the bottom: they want
     the most actionable thing.

Both are right. There's a third group: they printed it. Also
fine.

Wherever you started, the question I'd ask you right now is:

  "What step are you actually on?"

If you can name it (e.g., "I'm between Step 3 and Step 4"),
you're ahead of most buyers. If you can't, take 15 minutes
this week and figure it out. The whole roadmap is more useful
once you know your starting point.

- {{agent_first_name}}

P.S. The biggest delta I see in first-time buyers is between
Step 3 (savings) and Step 4 (lender pre-approval). If that's
where you are, the next email will be specifically about that
jump.
```

### Email N2 — Day 7

**Subject:** The Step 3 → Step 4 jump

```
Hey {{first_name}},

Most first-time buyers spend longer on Step 3 (building the
down payment + closing + reserve) than they need to, and
shorter on Step 4 (getting pre-approved) than they should.

Here's the trap.

Step 3 *feels* like the safe step. You're saving. You're
being responsible. You're not committing to anything. Easy
to stretch this for an extra 3 months while telling yourself
you're "not quite ready."

Step 4 *feels* scary. It involves talking to a lender. They'll
look at your credit. They'll ask questions about your debt.
There's a chance they tell you no. Easy to put this off
forever.

But here's the thing: Step 4 is the step that tells you
whether your Step 3 number is actually right. Most buyers I
work with think they need way more in the bank than they
actually do. Once we run their real numbers with a lender,
the savings target drops, and suddenly Step 5 (target areas)
is six weeks away, not six months.

This week, get one rate quote from one lender. That's it.
Just one. You don't have to commit to anything. You just need
a real data point about where you stand.

If you want a referral to a lender I trust for first-time
buyers in Orlando (including ones who work well with
newer-to-the-US credit profiles), reply to this email and
I'll send you 2 or 3 names. No pressure on what you do with
them.

- {{agent_first_name}}
```

### Email N3 — Day 14: Closing the series

**Subject:** When the roadmap turns into a conversation

```
Hey {{first_name}},

This is the last email in the roadmap series.

You've had it for two weeks now. You either picked it up and
ran with it, or you skimmed it once and life got in the way.
Both are normal. Most things sent over email get the second
treatment.

If you're still moving through it on your own, good. The
roadmap is everything I'd tell you in 30 minutes if you booked
a strategy session anyway. Use it.

If you're ready to talk through your specific situation
(your numbers, your target areas, your timeline), book a
30-minute call with me:

  → 30 minutes. Free. Video call.
  → Book it here: {{book_bss_link}}

The call is most useful when you're somewhere between Step 4
and Step 6 on the roadmap. If you're earlier than that, do
the work first; we'll cover more useful ground when you have
a lender letter in hand.

Either way, the auto-series ends here. I'll review your
contact this week and figure out the right next step from
here — most likely my monthly Orlando market update (one
email a month, no pitch).

- {{agent_first_name}}
```

After N3, the automation ends. The agent reviews the contact in Pipedrive and decides what's next (default: add to the `FTHB Monthly Market Update`).

---

## Interaction with the Tier automations

The Roadmap automation runs in parallel with whichever Tier automation the contact is in (if any). All four automations are linear and finite:

- **Tier A** (5 emails / 14 days)
- **Tier B** (3 emails / 5 days)
- **Tier C** (9 emails / 8 weeks)
- **Roadmap** (4 emails / 14 days)

Common parallel-run scenarios:

| Path into Roadmap | Most likely state | Entity on entry | Parallel automation running |
|---|---|---|---|
| LM1 Tier B → downloaded LM2 | `fthb_lm1_tier = NINETY_DAY` | Existing Deal updated | Tier B Deal automation (3 emails / 5 days; ends quickly) |
| LM2 standalone, no quiz history | `fthb_lm1_tier` unset | New Deal created | None |
| LM2 standalone → later took quiz to Tier A | `fthb_lm1_tier = READY_NOW` | Same Deal updated | Tier A Deal automation (5 emails / 14 days) |
| Tier C Lead → downloaded LM2 | `fthb_lm1_tier = FOUNDATION` | **Lead converted to Deal** | None on the new Deal (the Tier C automation enrollment was on the converted Lead) |

None of these scenarios require coordination between the automations. Each one runs its course. The agent reviews each contact at the end of whichever automation finishes last, and decides next steps (default: add to monthly newsletter).

The one edge worth naming: a contact who is currently `fthb_lm1_tier = FOUNDATION` (a converted-from-Lead Deal) will still hit the soft BSS CTA in Roadmap Email N3, which is technically a violation of the "never pitch BSS to Tier C from any sequence" rule. The mitigation is at the Make.com layer — the agent can configure Make.com to suppress Roadmap enrollment for these contacts if they want strict adherence. Iteration 1 enrolls everyone; tighten the routing later if it matters in practice.

---

## Implementation checklist

Before turning on the automation:

- [ ] Pipedrive custom fields exist **on both Lead and Deal entities** (per [04-readiness-filter-emails-v2.md](04-readiness-filter-emails-v2.md) checklist), including `fthb_received_lm2`, `fthb_lm2_received_at`, `fthb_lm2_source`.
- [ ] LM2-specific merge tags wired (resolving from the Deal): `fthb_roadmap_view_link`, `fthb_roadmap_pdf_link`, `fthb_readiness_quiz_link`, `book_bss_link`.
- [ ] All 4 email templates above (Email 0 + N1 + N2 + N3) saved as Pipedrive email campaigns the automation can reference.
- [ ] `FTHB LM2 - Roadmap` automation built in Pipedrive on the **Deal** entity: linear sequence of the 4 campaigns, no branching, no checks.
- [ ] Make.com scenario on the LM2 webhook: look up Person, apply conflict-resolution rules (always target Deal; convert any existing Lead with custom-field copy), write FTHB fields, enroll the Deal in the `FTHB LM2 - Roadmap` automation, return 200.
- [ ] Make.com Lead→Deal conversion logic explicitly copies all FTHB custom fields from the Lead to the new Deal before marking the Lead converted.
- [ ] Pipedrive view set up: "Deals whose Roadmap automation completed in the last 7 days" so the agent can batch end-of-automation review weekly (same review flow as the tier automations).
