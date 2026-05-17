
Every email here is final copy. Drop into Pipedrive Campaigns as-is. Curly braces `{{like_this}}` are Pipedrive merge fields that resolve against the Person record Make.com has just written.

The agent's email voice = the same as the result-page voice: direct, useful, no sales-y wind-up. Short subject lines. No exclamation points unless genuinely warranted.

---

## Email infrastructure choices baked into this sequence

- **From name:** `{{agent_first_name}} from Orlando Homes` (or whatever the agent's working brand name is, locked in Phase 1)
- **From email:** A real address the agent monitors. Replies are responded to within 24 hours. *No no-reply addresses.*
- **Sending tool:** **Pipedrive Campaigns** (the Campaigns addon on Pipedrive). All transactional + nurture email lives here as named campaigns; **Pipedrive Workflow Automations** enroll, unenroll, and pause contacts based on custom-field values (`fthb_lm1_tier`, `fthb_received_lm1`, `fthb_received_lm2`, etc.).
- **Make.com's role:** Receive the webhook, write the Google Sheet audit row, and create/update the Pipedrive Person with the right field values. Make.com does **not** send a single email itself — Pipedrive owns email entirely. This is the boundary; keep it clean.
- **Merge tags:** `{{like_this}}` in this doc maps to Pipedrive Campaigns merge fields. The agent will need to wire each one (`first_name`, `tier_label`, `display_score`, `fthb_result_page_link`, `book_bss_link`, `fthb_lm2_optin_link`, `fthb_retake_link`, `agent_first_name`, `agent_license_no`, `brokerage`) to the corresponding Pipedrive Person/Deal field before scheduling each campaign.
- **Tier reassignment rule:** If a contact retakes the scorecard and lands in a new tier, Make.com updates `fthb_lm1_tier` and sets `fthb_lm1_retaken_at` on the existing Pipedrive Person. A Pipedrive Workflow Automation listens for `fthb_lm1_tier` changes and:
  1. Unenrolls the contact from the campaign matching their old tier
  2. Enrolls them in the campaign matching the new tier, starting at email 1
- **Per-tier campaigns to build:** one Pipedrive Campaign per sequence below (`FTHB LM1 - Tier A`, `FTHB LM1 - Tier B`, `FTHB LM1 - Tier C`). Each transactional email (Email 0) is its own one-shot campaign or a Workflow-Automation-triggered template send; the agent can pick whichever Pipedrive Campaigns surface makes that easiest in practice.

---

## Email 0 (all tiers): Transactional delivery

**Trigger:** Immediately on form submit (within 60 seconds).
**Goal:** Confirm receipt, deliver the result link, set the tone.

**Subject:** Your Orlando readiness score: {{tier_label}}
**Preview text:** Score, timeline, and the 2 mistakes to avoid. All inside.

```
Hey {{first_name}},

You came back as {{tier_label}}, {{display_score}}/100.

Here's the full result with your tier explanation, the 2 mistakes
buyers in your tier most often make, and a 1-page Orlando market
snapshot:

    {{fthb_result_page_link}}

The result page also has your specific next step at the bottom.

Two things to know:

1. I read every reply. If anything in the result page raises a
   question (about your score, the market data, the next step,
   anything), just hit reply.

2. I'm not going to call you, text you, or pass your info to
   anyone. Not now, not ever. If we end up talking, it'll be
   because you booked something on your own.

Talk soon,
{{agent_first_name}}
{{agent_license_no}} | {{brokerage}}

P.S. Bookmark the result-page link. Nothing on it expires. The
2-mistakes section gets a lot more useful the day before you
actually talk to a lender, so come back to it then.
```

---

## Tier A: "Ready Now" sequence (5 emails)

Cadence: Email 1 immediately after the transactional. Then days 2, 5, 9, 14.
Goal: Get the Buyer Strategy Session booked. They're already qualified; this sequence creates urgency without manufacturing it.

### Email A1 - Day 0 (sent ~10 minutes after transactional)

**Subject:** One thing about your "Ready Now" score

```
Hey {{first_name}},

Quick follow-up on your scorecard result.

"Ready Now" doesn't just mean you can technically qualify for a
mortgage. It means the gap between you and keys is short enough
that timing matters in a way it doesn't for other tiers.

Here's the thing nobody tells first-time buyers in your spot:

The two months between pre-approval and closing are the months
where deals fall apart the most. Not because of the market,
but because of avoidable mistakes the buyer makes (new credit,
large deposits, job changes).

The Buyer Strategy Session I offer is built for exactly this
window. We go through:

- Your pre-approval (is it actually strong enough to win in your
  target neighborhoods, or just "good enough"?)
- 2–3 target areas and what's actually available right now
- The 3 offer mechanics that consistently win along the I-4
  corridor (Sanford to Downtown, both Seminole and Orange side)
  without overpaying

30 minutes. Free. Zoom or in person. No pitch. If we're a fit,
great; if not, you keep the action plan.

    {{book_bss_link}}

- {{agent_first_name}}
```

### Email A2 - Day 2

**Subject:** Seminole County vs. Orange County: a 90-second read

```
Hey {{first_name}},

Most "Ready Now" buyers in Orlando default to one of two
paths:

  Path 1: Buy on the Orange side (Maitland, Winter Park, College
  Park, the I-4 side of Apopka) — closer to Downtown,
  walkability or commute wins, but Orange school zoning is more
  uneven block to block

  Path 2: Push north on the Seminole side (Altamonte Springs,
  Longwood, Lake Mary, Sanford) — tighter inventory in your
  range, longer Downtown commute, but the strongest school-
  zoning resale floor in the metro

There's a third path most people miss: the **Seminole/Orange
border zone** — Casselberry and southern Altamonte Springs
(Seminole) sitting right against Maitland and northern Winter
Park (Orange). Specific blocks let you live within a 10-minute
drive of both county school systems and choose your trade-off
street by street, with meaningful price-per-square-foot
differences between the two sides.

Worth 30 minutes to look at the map with you and see if any of
your target areas qualify.

    {{book_bss_link}}

- {{agent_first_name}}
```

### Email A3 - Day 5

**Subject:** The pre-approval question I always ask first

```
Hey {{first_name}},

When someone in the "Ready Now" tier reaches out, the very first
question I ask is:

"Did your lender pre-approve you, or did they pre-qualify you?"

These sound the same. They aren't.

  - Pre-qualification = you told them your numbers and they did
    quick math. No documents pulled. No underwriting. Useless on
    a competitive offer.

  - Pre-approval = they pulled credit, verified income, verified
    assets, and ran it past an underwriter. Real.

If you have a letter, look at it now. If it doesn't say
"pre-approved" (and most don't), that's the first thing to fix
before you make an offer on anything.

If you're not sure, send me a redacted copy of your letter and I
can tell you in 5 minutes.

    Reply with the letter, or → {{book_bss_link}}

- {{agent_first_name}}
```

### Email A4 - Day 9

**Subject:** The one contingency that does the heavy lifting

```
Hey {{first_name}},

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

This is the kind of thing we cover on the strategy session.
How to actually use your contingencies and what your offer
should be doing for you.

    {{book_bss_link}}

- {{agent_first_name}}
```

### Email A5 - Day 14

**Subject:** Last note from me

```
Hey {{first_name}},

I'm not going to keep emailing you about the strategy session.

If you want to talk, the link is below. If not, totally fine.
You have the result page and you have the action items from
the last few emails. You can absolutely do this on your own.

If you go your own way and run into a wall, my inbox is open. No
hard feelings, no awkwardness. Just reply.

    {{book_bss_link}}

- {{agent_first_name}}

P.S. If a friend in Orlando is at the same stage you are, the
scorecard link is open: {{fthb_readiness_quiz_link}}. No referral
fee, no tracking. I'd just rather more first-time buyers know
where they actually stand before they start shopping.
```

After A5, the contact moves to the **monthly market-update list** (same as the Tier C long-term nurture; see below).

---

## Tier B: "90-Day Sprint" sequence (6 emails)

Cadence: Email 1 ~10 min after transactional. Then days 2, 4, 7, 12, 21.
Goal: Get them to consume LM2 (the 9-Step Roadmap), then convert to BSS. This is the **highest-volume tier** and where the agent's nurture has the most impact.

### Email B1 - Day 0

**Subject:** Your 90-day game plan: start here

```
Hey {{first_name}},

You're in the 90-Day Sprint tier, which means the next 90 days
are the whole game.

On the result page, your "what's next" was to grab my 9-Step
First Home Roadmap. If you didn't snag it yet, here it is:

    {{fthb_lm2_optin_link}}

It's the exact step-by-step process between "I think I'm ready"
and keys in your hand in Orlando. Every step shows what you do,
what your agent does, and how long it takes.

Three things I built into it that you won't find in a generic
buyer guide:

  - The 3 places first-time buyers in Orlando lose money (named,
    specific, avoidable)
  - HOA disclosure timelines and Florida rainy-season inspection
    timing (local stuff)
  - The one number on your credit report that affects your rate
    more than your score does

Read it. Then we can talk.

- {{agent_first_name}}
```

### Email B2 - Day 2

**Subject:** Before you look at a single house

```
Hey {{first_name}},

If you skipped the roadmap from yesterday, here's the single
biggest mistake I see Sprint-tier buyers make:

They start going to open houses before they have a pre-approval.

I get it. Open houses feel productive. They aren't, not in
this tier. Here's what actually happens:

  1. You walk through 4 houses on a Sunday.
  2. You fall in love with one.
  3. You go to a lender on Monday.
  4. The lender tells you that house was $40K above what you'd
     comfortably qualify for.
  5. You spend the next 3 weeks comparing every house you see to
     the one you can't have.

The fix is to do those steps in the other order. Get the
pre-approval letter first. Then walk through houses in your
actual range. You will see houses you like, I promise.

The full sequence (with the lender shortlist, the documents to
gather, and the question to ask every lender to vet them) is in
the roadmap:

    {{fthb_lm2_optin_link}}

- {{agent_first_name}}
```

### Email B3 - Day 4

**Subject:** When to use the builder's lender

```
Hey {{first_name}},

If new construction is on your radar at all (Apopka, Lake Mary,
Sanford), here is a question most first-time buyers do not run
the numbers on: should you use the builder's preferred lender?

The instinct is to say no. Builder's lenders have a reputation
for marking up rates to offset incentives. In 2026 that
reputation is often wrong.

Builders are sitting on inventory and competing hard, which
means many of them are subsidizing mortgages down to rates that
genuinely beat outside lenders. It is common right now to see a
builder rate at 5.49% or 5.99% while resale rates sit at
6.75-6.99%. On a $300K loan, that spread saves you $150-$290 a
month, and tens of thousands across the life of the loan.

The catch: some of those advertised rates are temporary
buydowns (2-1 or 3-2-1) that reset to the note rate after year
1, 2, or 3. Year 1 looks great. Year 4 can be a $400-$600
monthly jump you were not budgeting for.

How to handle it:

  1. Get a real quote from at least one outside lender on the
     same loan size.
  2. Ask the builder's lender, in writing, whether the
     advertised rate is permanent or a temporary buydown. If
     it's a buydown, get the effective rate for each year
     separately.
  3. Compare three numbers: year 1 payment, year 4 payment
     (after a typical buydown resets), and total interest paid
     over 7 years (a realistic first-home hold period).
  4. Treat closing-cost concessions and finish-package credits
     as separate negotiable items, not part of the rate
     decision.

Step 4 (Lender Selection) in the roadmap walks through both the
new-construction and resale paths:

    {{fthb_lm2_optin_link}}

- {{agent_first_name}}
```

### Email B4 - Day 7

**Subject:** Quick question for you

```
Hey {{first_name}},

Where are you actually getting stuck right now?

I've sent the roadmap and a couple of specific lessons. Most
Sprint-tier buyers I talk to are stuck on one of three things:

  1. Credit: they're afraid to look at their report, so they
     don't.
  2. Down payment: they're not sure how much they really need,
     or where to put the money in the meantime.
  3. Choosing a lender: they don't know who to call or what to
     ask.

If you're stuck on one of these, just reply with the number
(1, 2, or 3) and I'll send the most useful thing I've got on
that specific block.

- {{agent_first_name}}

P.S. This isn't a scripted auto-thing. You reply, I read it.
```

### Email B5 - Day 12

**Subject:** When the Sprint becomes Ready Now

```
Hey {{first_name}},

Here's how you know you've crossed from Sprint to Ready Now,
which usually happens at the 60-day mark for buyers in your
tier if they're doing the work:

  1. You have a real pre-approval letter (not pre-qualification)
     from a lender you trust.
  2. Your savings account has down payment + 3% closing + a
     30-day reserve.
  3. You know your two target areas and roughly what's
     available in your range there.

If you can check all three, you're a Sprint graduate, and
that's when the Buyer Strategy Session actually pays off:

    {{book_bss_link}}

If you're not there yet, that's totally normal. Keep going.

- {{agent_first_name}}
```

### Email B6 - Day 21

**Subject:** Last note (then I back off)

```
Hey {{first_name}},

I'm not going to keep emailing about the BSS. Either you're
making progress on the 90-day work or you're not, and either
way more email from me doesn't help.

What I'll do instead: I'll keep you on the monthly market
update. One email a month, Orlando-specific. That's it.

If you want to talk before then, link is here whenever you're
ready:

    {{book_bss_link}}

- {{agent_first_name}}
```

After B6, the contact moves to the **monthly market-update list**.

---

## Tier C: "Foundation Phase" sequence (long-form nurture, every 2 weeks)

Cadence: Bi-weekly, indefinite. They unsubscribe or graduate up a tier when they retake.
Goal: Stay useful, stay top-of-mind, do not pitch the BSS until they cross into Tier B/A. The bar for every email is *"is this useful to someone 6–12 months out from buying?"*

### Email C0 - Day 0 (welcome)

**Subject:** What "Foundation Phase" actually means

```
Hey {{first_name}},

Foundation Phase doesn't mean "no." It means "not yet, and here
is what to do with the time."

You're on my Foundation list now. Here's what to expect:

  - One short email every other week
  - 100% useful, 0% sales pitch
  - Topics: credit, savings, market data, things to avoid

When you cross into the 90-Day Sprint tier (which usually means
your credit, savings, and timeline all line up), I'll let you
know. You can retake the scorecard anytime to check:

    {{fthb_retake_link}}

First real email comes in about 2 weeks.

- {{agent_first_name}}
```

### Email C1 - Day 14: Credit

**Subject:** The one place to actually pull your credit (and why)

```
{{first_name}},

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

### Email C2 - Day 28: Savings

**Subject:** Where to actually park your home fund

```
{{first_name}},

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

Bonus move: name the account "Orlando Home Fund." Sounds dumb.
Works. It's measurably harder to pull money out of an account
with a goal name on it.

- {{agent_first_name}}
```

### Email C3+ - Subsequent topics (rotate every 2 weeks)

Continue the bi-weekly cadence using these topics, in order. Each email follows the same structure: one concrete idea, no pitch, 200–300 words.

1. Down payment myths: the 20% myth, real minimums by loan type
2. Why your DTI matters more than your score for first-time buyers
3. Florida-specific: FHA vs. conventional for Orlando first-timers
4. Hidden costs at closing (line item by line item; what each fee really is)
5. Why HOA dues should be in your "monthly cost" math, not separate
6. The single most useful Florida assistance program for FTHBs (current program TBD; agent updates)
7. Why renewing your lease for 12 months is fine, and when it's not
8. The "stretch" trap: how to know if a payment is too much
9. Tax implications of homeownership in Florida (it's mostly good news, but specific)
10. Building 6 months of "homeowner's reserve": what it is, why lenders love it

After topic 10, loop back to topic 1 with refreshed examples and market data.

---

## Unsubscribe / preference handling

Every email has the standard Pipedrive Campaigns unsubscribe footer (required for CAN-SPAM compliance and provided by the Pipedrive Campaigns template). Additionally:

> *Don't want these every other week? Reply "monthly" and I'll switch you to once a month. Reply "stop" and I'll take you off entirely.*

The standard unsubscribe link removes them from all Pipedrive Campaigns. The "monthly" reply is handled manually by the agent (move the contact's `nurture_cadence` field to `monthly`; a Pipedrive Workflow Automation routes them to the slower cadence). The "stop" reply triggers the standard unsubscribe in Pipedrive. Both work without code on the static site.
