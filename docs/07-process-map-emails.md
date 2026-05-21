Same email rules as LM1: real from-address, no no-reply, replies answered within 24 hours. All sending happens in **Pipedrive Campaigns**, triggered by **Pipedrive Workflow Automations** that fire off the custom fields Make.com sets on the contact (see `05-process-map-spec.md`). Curly braces are Pipedrive merge fields.

LM2's nurture is **shorter and more targeted** than LM1's. The reader has already opted into LM1 (in most cases, Tier B), already gotten useful content, and is now consuming the deepest free asset in the funnel. The job of these emails is to make the BSS feel like the obvious next step _only when they're ready for it_.

**Build this as one Pipedrive Campaign:** `FTHB LM2 - Roadmap`. Three nurture emails on the cadence below, plus Email 0 as the transactional. The campaign is the same for both `source` values because, per the spec, the roadmap content level-sets all readers.

---

## Email 0 (LM2): Transactional delivery

**Trigger:** Immediately on form submit, both `fthb_lm1_tier_b` and `fthb_lm2_standalone` sources.
**Goal:** Deliver the PDF and the web link. Set expectations for follow-up.

**Subject:** Your 9-Step Orlando Home Roadmap (link + PDF inside)
**Preview text:** The full roadmap, plus where to start reading depending on where you are.

```
Hey *|FIRST_NAME|*,

Here's the 9-Step First Home Roadmap as promised.

  → Read it on the web: {{fthb_roadmap_view_link}}
  → Download the PDF (for offline reading or printing):
    {{fthb_roadmap_pdf_link}}

If you took the Readiness Scorecard and landed in the 90-Day
Sprint tier, **start at Step 4** (lender pre-approval). That's
the bottleneck for almost everyone in that tier.

If you came in from one of my posts about the home-buying
process, start at the visual roadmap and find the step that
looks most like where you are.

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

---

## LM2 Nurture Sequence (3 emails)

Cadence: Email 1 on Day 3 (gives the reader time to consume). Then Day 7. Then Day 14.

The sequence is the same regardless of source (`fthb_lm1_tier_b` or `fthb_lm2_standalone`); the roadmap content level-sets all readers.

### Email N1 - Day 3

**Subject:** Which step did you start on?

```
Hey *|FIRST_NAME|*,

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

### Email N2 - Day 7

**Subject:** The Step 3 → Step 4 jump

```
Hey *|FIRST_NAME|*,

Most first-time buyers spend longer on Step 3 (building the
down payment + closing + reserve) than they need to, and
shorter on Step 4 (getting pre-approved) than they should.

Here's the trap.

Step 3 *feels* like the safe step. You're saving. You're being
responsible. You're not committing to anything. Easy to stretch
this for an extra 3 months while telling yourself you're "not
quite ready."

Step 4 *feels* scary. It involves talking to a lender. They'll
look at your credit. They'll ask questions about your debt.
There's a chance they tell you no. Easy to put this off forever.

But here's the thing: Step 4 is the step that tells you whether
your Step 3 number is actually right. Most buyers I work with
think they need way more in the bank than they actually do.
Once we run their real numbers with a lender, the savings
target drops, and suddenly Step 5 (target areas) is six weeks
away, not six months.

If you're somewhere between Step 3 and Step 4 right now: this
week, get one rate quote from one lender. That's it. Just one.
You don't have to commit to anything. You just need a real
data point about where you stand.

If you want a referral to a lender I trust for first-time
buyers in Orlando (including ones who work well with newer-
to-the-US credit profiles), reply to this email and I'll send
you 2 or 3 names. No pressure on what you do with them.

- {{agent_first_name}}
```

### Email N3 - Day 14

**Subject:** When the roadmap turns into a conversation

```
Hey *|FIRST_NAME|*,

This is my last note on the roadmap.

You've had it for two weeks now. You either picked it up and
ran with it, or you skimmed it once and life got in the way.
Both are normal. Most things sent over email get the second
treatment, including stuff that would help.

If you're still moving through it on your own. Good. The
roadmap is everything I'd tell you in 30 minutes if you booked
the strategy session anyway. Use it.

If you're ready to talk through your specific situation
(your scorecard tier, your numbers, your target areas), book
a 30-minute call with me:

  → 30 minutes. Free. Video call.
  → Book it here: {{book_bss_link}}

The call is most useful when you're somewhere between Step 4
and Step 6 on the roadmap. If you're earlier than that, do
the work first; we'll cover more useful ground when you have
a lender letter in hand.

Either way: I'll stop emailing you about the roadmap after
this. You'll get my monthly Orlando market update from here
on, and that's it.

Good luck with the next 90 days.

- {{agent_first_name}}
```

After N3, the contact moves to the **monthly market-update list** (same as Tier A and Tier B graduates).

---

## Tier B contacts: avoiding the email storm

Anyone arriving at LM2 via `source = fthb_lm1_tier_b` is already enrolled in the **LM1 Tier B campaign** in Pipedrive. Without careful routing, they'll get hammered with two parallel sequences.

The rule: **the moment Make.com flips `fthb_received_lm2 = true` on a contact who has `fthb_lm1_tier = NINETY_DAY`, unenroll them from the LM1 Tier B campaign and enroll them in the LM2 campaign.**

Split of responsibilities:

**Make.com (on `magnet: "fthb_lm2"` + `source: "fthb_lm1_tier_b"`):**

1. Webhook receives the payload
2. Write the Google Sheet audit row
3. Look up the Pipedrive Person by email
4. Update the Person: `fthb_received_lm2 = true`, `fthb_lm2_received_at = now`, language toggle if changed
5. Return `200`

**Pipedrive Workflow Automation (on `fthb_received_lm2` flipped to `true`):**

1. Unenroll the contact from the `FTHB LM1 - Tier B` campaign (cancels all scheduled future emails for that contact in that campaign)
2. Enroll the contact in the `FTHB LM2 - Roadmap` campaign starting at Email 0
3. The campaign's built-in scheduling drips N1, N2, N3 on Day 3, Day 7, Day 14

A contact who never opts into LM2 stays in the LM1 Tier B campaign and finishes it normally; no automation fires.

---

## What goes in the monthly market-update list

Every contact eventually lands here, regardless of which sequence they came from. This is the long-tail nurture: once per month, Orlando-specific, useful.

Each monthly email includes:

- **One short market data update:** median price for one Seminole or Orange County area, with what the move means for FTHBs in that area
- **One useful tip:** financing, inspections, neighborhood research, etc. (rotating from the same topic list as the Tier C nurture)
- **One soft CTA:** never the BSS by default. Examples:
  - "Want to retake the scorecard? Your numbers may have moved."
  - "Want the roadmap if you haven't seen it yet?"
  - "Replying to this email is the fastest way to get a question answered."

The BSS gets pitched in the monthly email **once per quarter** (in March, June, September, and December), and only as a "P.S. If you want to talk this quarter, here's the link."
