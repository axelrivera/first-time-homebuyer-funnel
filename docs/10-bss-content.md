## What this doc is

The shipping copy and template structures for the **Buyer Strategy Session**. The offer spec ([09-bss-offer-spec.md](09-bss-offer-spec.md)) defines what the BSS is and how it routes; this doc defines what the prospect actually reads on the landing page, fills out on the intake form, hears on the call, and receives in the deliverables. Math and shortlist logic are documented separately in [12-bss-math-and-shortlist.md](12-bss-math-and-shortlist.md).

Status: **DRAFT**. Every block here needs a Hormozi voice pass (the "feel stupid to say no" + "getting a deal, not being sold to" tests from [CLAUDE.md](../CLAUDE.md)) before it goes live. The agent's name, license number, and brokerage are merge fields, not hard-coded text.

---

## v0 MVP cut — what from this doc ships first

Per the v0 scope in [09-bss-offer-spec.md](09-bss-offer-spec.md), the BSS launches as a manual, low-automation offer. From this doc:

- **Ships as written**: the landing page (all sections, hero through FAQ), the in-call script (all 5 blocks), the Shortlist PDF template, and the math sheet's user-facing copy.
- **Trims to 4 questions in v0**: the intake form. Keep Q1 (partner Y/N), Q2 (timeline), Q4 (cash bucket), and Q5 (neighborhoods considering). Defer Q3 (down-payment source; the agent asks it first in Block 2 anyway) and Q6 (the catch-all free-text field; the diagnostic block surfaces this live). The full 6-question form is the v1 target.
- **Deferred to v1**: the **Lender Comparison Card** PDF and the **Red-Flag Property Filter** PDF. In v0, both are delivered conversationally. The agent names the seven red flags verbally in Block 4 and sends a plaintext follow-up email with three lender contacts (name, direct phone, direct email, one-line "best for" tag). Produce the printed PDFs once the agent has heard which lender attributes and which red flags actually land.

Treat the **Lender Comparison Card** and **Red-Flag Property Filter** sections below as v1 design reference, not v0 ship targets. The intake form section below documents the full 6-question form; in v0, drop Q3 and Q6.

---

## Public-facing name

> **The Orlando First-Time Buyer Strategy Session**
>
> *30 Minutes to Your Personal Buy-Box, a Shortlist of Orlando Neighborhoods That Fit Your Numbers, and the Exact Next Step. Whether You Hire Me or Not.*

The subhead is proposed, not locked. The locked LM1 and LM2 titles in [CLAUDE.md](../CLAUDE.md) follow the same template: a specific result plus a time-to-result. Once this wording locks, it must appear verbatim on the landing page, every email subject line, the calendar tool description, and any DM script.

---

## Landing page

Route: `/orlando-first-time-buyer-strategy-session`. Linked from: the LM1 Tier A result page, the LM2 viewer page, Tier A nurture emails A1 through A5, the LM2 transactional email, and any warm-outreach DM the agent sends.

### Hero block

**Headline:**
> Know What You Can Buy, Where to Buy It, and What It Will Cost. In 30 Minutes.

**Subhead:**
> A free one-on-one call. You leave with a written buy-box, a neighborhood shortlist that fits your actual numbers, and a clear next step. No pitch, no list of houses to tour.

**Primary CTA button:**
> Book My 30-Minute Buyer Strategy Session

The button is the only CTA in the hero block. No secondary link, no "call me directly" alternate. One door.

### What you walk out with (above the fold on desktop, below the hero on mobile)

Four cards. Each is one of the four bonuses from [09-bss-offer-spec.md](09-bss-offer-spec.md), rewritten in landing-page voice.

1. **A 3-Neighborhood Shortlist, built around your numbers.**
   Three Orlando ZIP codes I'd send you to look at this week, given your cash, your credit, your commute, and what you said you cannot live without. If you walk in with neighborhoods already in mind, those lead the list and the math runs on them first. Sent to you as a PDF within 24 hours of our call.

2. **The Live Payment Math, on your specific numbers.**
   Built on the call in front of you in a Google Sheet you keep. Today's rate, your down payment, FHA vs conventional, the Florida tax and insurance lines most calculators forget. You see the math; you don't take my word for it. The link is yours after the call so you can re-run the numbers as your cash grows or rates move.

3. **The Red-Flag Property Filter.**
   Seven things first-time buyers in this part of Orlando lose money on, from failed-permit additions to roof age versus insurance carrier appetite to school-zoning math on resale. A 1-page PDF you read before every showing.

4. **Three lenders, named, with phone numbers.**
   Not a referral form. Three lenders I have personally worked with, with a one-line "best for" tag each (credit range, FHA specialist, fastest close). I do not collect a fee from any of them. You leave the call with their direct contact, not a "someone will reach out."

### What the 30 minutes actually looks like

A short, honest preview of the agenda. The reader should finish this section knowing what they are signing up for.

- **Minutes 0 to 3** — I tell you what I already know from your scorecard, you tell me whether to start by asking you about neighborhoods or by running your numbers first. You pick.
- **Minutes 3 to 10** — Five quick diagnostic questions to fill the gaps. Down-payment source, household, commute, what you will not compromise on, your backup plan if you don't buy in 6 months.
- **Minutes 10 to 20** — We open the math sheet. Your numbers, today's rate, real Florida tax and insurance. I show you what a $360,000 home actually costs to own. We toggle FHA, conventional, different down payments. You watch.
- **Minutes 20 to 27** — Map up. Three ZIPs along the Sanford to Downtown Orlando route. If you brought neighborhoods of your own, we run those first and I add one or two you may not have considered. You leave with comps on real addresses.
- **Minutes 27 to 30** — Three options for what happens next. Option 1: I send you the shortlist PDF in 24 hours and you take it from here. Option 2: I introduce you to one of the lenders we discussed, free pre-approval call. Option 3: I become your buyer's agent and we go tour the top two next weekend. Pick whichever fits. The shortlist is yours either way.

### The 30-Minute Promise (guarantee)

Render as a callout box, not a paragraph.

> **If after our 30 minutes you don't walk away with (1) a written buy-box, (2) a personalized neighborhood shortlist, and (3) a clearer next step than you had when you booked, I'll send you the shortlist anyway and personally introduce you to one of the three lenders for a free pre-approval call.**
>
> No commitment to use me as your agent. Ever.

### Frequently asked

Five questions. Short answers. No marketing voice.

1. **Is this really free?** Yes. There is no upsell on the call and no charge afterward. If you want me as your agent, that is a separate conversation we have only if you ask for it.

2. **Will you pressure me to sign anything on the call?** No. The buyer's representation agreement is option 3 of three. Option 1 is "send me the shortlist, that's it." You can pick option 1 and never hear from me again. That is fine.

3. **Do I need a pre-approval already?** No. Most prospects do not. The math we do on the call is what tells us whether you are ready to talk to a lender at all, and which lender to talk to. If you do already have a pre-approval, bring it.

4. **What if I already have a neighborhood in mind?** Bring it. The shortlist defaults to three I'd pick, but if you walk in with one or two in mind, those go to the front of the list and the math runs on them first. I might add an alternate you have not considered, but your preferences lead.

5. **What if I am not in Orlando yet?** Fine. Most relocation buyers I work with start their search 60 to 120 days out. We can do this call before you have moved.

### Below-the-fold CTA

Same button, same wording, same destination. No alternate.

> Book My 30-Minute Buyer Strategy Session

Followed by one line of trust copy:

> {{agent_first_name}} {{agent_last_name}}. {{brokerage}}. License {{agent_license_no}}.

---

## Intake form

Sent in the booking confirmation email and linked in the calendar invite. Designed to take less than 3 minutes. Submission is optional but the form copy nudges the prospect into completing it.

### Form preamble (above the first question)

> The more I know before we get on the call, the less of our 30 minutes we spend on warm-up. Six questions, none of them require uploading a document or sharing a credit score. Hit skip on anything you don't want to answer.

### Questions

1. **Partner or co-buyer?**
   - *Type:* Yes / No. If yes, name field appears.
   - *Why:* If yes, they should be on the call. The form copy says so: "If yes, please bring them. The math we do only works for both of you if both of you watched it happen."

2. **When do you want to be in a home?**
   - *Type:* Single-select. *0 to 3 months / 3 to 6 months / 6 to 12 months / 12+ months / honestly not sure*.
   - *Why:* Calibrates urgency, lender pick, and whether the call leans more "act now" or "build the plan."

3. **Where is your down payment coming from?**
   - *Type:* Multi-select. *Savings I already have / Gift from family / 401k withdrawal or loan / Sale of a current home / Not sure yet*.
   - *Why:* Each source has specific gotchas. Gift funds need a paper trail. 401k withdrawals have a tax bite. The math block on the call goes deeper if the agent already knows.

4. **Roughly how much cash do you have available for the deal?**
   - *Type:* Bucket select. *Under $10k / $10k to $25k / $25k to $50k / $50k+ / I'd rather discuss on the call*.
   - *Why:* Calibrates the price ceiling we run the math against. The bucket is on purpose; nobody types a precise number into a form.

5. **Any neighborhoods you're already considering?**
   - *Type:* Free text, optional.
   - *Form copy under the field:* "If you have specific ZIPs, towns, or areas you've been looking at, tell me. They'll lead our shortlist and the math will run on them first. If you don't have any in mind, leave this blank and I'll bring three to the call."
   - *Why:* This is the load-bearing input for the shortlist branch (Path A, B, or C in [12-bss-math-and-shortlist.md](12-bss-math-and-shortlist.md)). Capturing it pre-call lets the agent pre-load the math sheet with the right ZIPs and have at least one comp ready.

6. **Anything you want me to know before we talk?**
   - *Type:* Free text, optional.
   - *Form copy under the field:* "Top must-haves, hard deal-breakers, life context, recent quotes you've gotten from other agents or lenders. Whatever helps. Skip if nothing comes to mind."
   - *Why:* Catches the things the buckets miss. Stored in Pipedrive notes; not used for routing.

### Form submit confirmation copy

Inline thank-you after submission, no redirect.

> Got it. I'll read this before we talk so we can get to the math fast. See you at {{scheduled_time}}.

If the prospect tries to close the page before submitting, no exit-intent popup. Friction is a tell that the prospect was already at the limit of their patience; chasing them out costs more than the form would have gained.

---

## In-call script

Block-by-block dialogue. The agent does not memorize this; the agent reads through it twice during call prep and then has the math sheet and the Pipedrive contact card open. Italics are the agent speaking. Roman type is stage direction.

### Block 1: Welcome and frame (0:00 to 3:00)

Agent has the prospect's LM1 scorecard answers open in Pipedrive and the math sheet pre-loaded with the intake form values already typed in. Camera on.

> *"Hey {{first_name}}, thanks for booking. I'm {{agent_first_name}}. Before we start, three quick things."*
>
> *"One. I've already read your scorecard and your intake. So we don't have to redo that. You came back as Tier {{tier_letter}}, which means {{tier_meaning_in_one_sentence}}, and you said your cash is in the {{cash_bucket}} range and you're targeting {{timeline}}. If I got any of that wrong, stop me now."*

Pause. Wait for a yes or a correction. If correction, update the math sheet live so the prospect sees the change happen.

> *"Two. The goal of the next 30 minutes is to hand you a buy-box, a shortlist of neighborhoods, and a clear next step. I am not going to ask you to sign anything today. The shortlist gets emailed to you as a PDF within 24 hours either way."*
>
> *"Three. Before I show you neighborhoods, two ways we can do this. Either you tell me where you're already looking and I'll run the math on those places first. Or I show you three I'd pick from your numbers and you tell me what's missing. Which one?"*

Wait for the answer. This branches the call. Make a note in the math sheet's B14 cell (the agent updates the cell live).

### Block 2: Diagnostic (3:00 to 10:00)

Five questions the LM1 form did not cover. Move fast; the agent should be filling cells in the math sheet as the prospect answers.

> *"Quick five. First, down payment source. You said {{down_payment_source}} on the form. Is that all of it, or some of it?"*

Listen for: gift funds (need paper trail and a signed letter), 401k loan vs withdrawal (very different tax outcomes), home sale contingent (timing risk).

> *"Second, household. Who's living in the home? Anyone we should plan around now or in the next five years (kids, parents, room for a home office that has to be a separate room)?"*

> *"Third, commute. Where are you driving to most days, and what's the maximum tolerable one-way time?"*

> *"Fourth, the one thing you will not compromise on. Not the wish list. The one thing where if the house doesn't have it, you walk."*

> *"Fifth, the backup plan. If you don't buy in the next six months, what happens? Lease renewal? Stay with family? Job-relocation deadline?"*

Note the backup-plan answer aloud, then transition.

> *"Okay. Math sheet time. Sharing my screen."*

### Block 3: Live math (10:00 to 20:00)

Screen share starts. The math sheet is on the Inputs tab.

> *"This is what we'll work in. Your numbers go up here, the math falls out below. I'm typing your stuff in real time, so if you see anything that's off, say so. Round numbers are fine."*

Walk through each input cell, narrating. Prospect should be able to see what the agent is typing.

> *"Combined gross monthly income. You said roughly {{income_bucket}}. Going with {{midpoint_or_explicit_number}}; tell me if you want to use a different number."*
>
> *"Existing monthly debts. Car payments, student loans, credit-card minimums. I am skipping anything you pay off in full each month. What's that total?"*

Continue through credit band, cash available, target price, term, property type, county, expected HOA, today's rate.

When all inputs are in, switch to the **Loan Scenarios** tab.

> *"Three scenarios side by side. FHA at 3.5% down. Conventional at 5% down. Conventional at 20% down. You can see the monthly payment, mortgage insurance on its own line so we don't bundle it, the property tax for {{county}}, Florida homeowners insurance, and HOA if any."*
>
> *"This number here. That's the all-in monthly. This number here. That's the cash you actually need to bring to closing. And this one. That's what you have left over after you sign. The bottom line is what tells us whether buying right now keeps you safe or makes you house-poor."*

Pause. Let the prospect look. Wait for the question. The first question is almost always about insurance or property tax; both are higher than what most calculators show.

> *"Yeah, the insurance number surprises everyone the first time. Florida is in a hard market for homeowners insurance right now. That's a real number for a {{property_type}} {{age_band}} in {{county}}. We will get you a real quote with the lender before any offer goes in, but this is the right band to plan against."*

Toggle scenarios live. Show what changes when the prospect uses FHA vs conventional. Show what changes if they wait 6 months and grow their down payment to the next bucket. Show what changes at a target price $20,000 lower or higher.

> *"Last thing on this tab. Reserves after close. If this number is yellow, that's one month or less of all-in housing payment sitting in the bank the day after you sign. If it's red, that's less than half a month. Either color means we either lower the price target or we wait three months and grow the cash side. Anyone telling you otherwise is selling you a home, not helping you buy one."*

Transition.

> *"Okay. We have a number. Let's map it."*

### Block 4: Shortlist walk-through (20:00 to 27:00)

This is the block that branches on the answer from Block 1. The agent has the **Shortlist Math** tab on the math sheet and a map (Zillow or MLS) on a second tab or window.

#### Path A: Prospect had no neighborhoods in mind

> *"Three ZIPs along the Sanford to Downtown Orlando line that fit your number. Going to start with the most comfortable, then the stretch, then one you probably haven't thought about."*

Walk through each. Real comp on the map, real price, real days on market. Two sentences per ZIP on what it gives them and what it costs.

#### Path B: Prospect named neighborhoods, and the math works

> *"You said {{prospect_zip_1}} and {{prospect_zip_2}}. Let's run them first."*

Plug the median price for {{prospect_zip_1}} into the math sheet. Show the payment.

> *"At {{prospect_zip_1}}, here's your number. Payment lands at {{X}}, cash to close at {{Y}}, reserves at {{Z}}. That works for your numbers."*

Same for {{prospect_zip_2}}. Then add one alternate.

> *"Want me to add one you probably did not ask about? {{wildcard_zip}}. Here's why I think it fits."*

#### Path C: Prospect named neighborhoods, and the math does not work

> *"You said {{prospect_zip_1}}. Want me to be honest with you?"*

Wait for yes.

> *"At your current cash and rate, {{prospect_zip_1}} runs at {{X}} a month all-in, your reserves are negative the day after close, and your back-end DTI sits at {{Y}}. That's not a number you should walk into."*
>
> *"I can show you three things that change this picture. You wait six months and grow the cash side. You stretch to FHA at a smaller home in the same ZIP. You keep the price and slide one ZIP outward where the same square footage costs {{Z}} less. The shortlist I send you will have all three, written down. You decide."*

Show each on the math sheet, live.

### Block 5: Next step and close (27:00 to 30:00)

Camera back if it was off during screen share.

> *"Three minutes left. So here's how this works. Three options."*
>
> *"Option 1. I send you the shortlist PDF in 24 hours. The math sheet link is already in your inbox. You take it from there. No follow-up from me. The shortlist is yours."*
>
> *"Option 2. I introduce you to one of the three lenders we just talked about for a free pre-approval call. I email them with you copied today or tomorrow. You decide what happens after that conversation."*
>
> *"Option 3. I become your buyer's agent. We sign the buyer's agreement. We go tour the top two on the shortlist next weekend. That one has paperwork. It does not have to be today."*
>
> *"Either way works for me. Pick whichever fits."*

Wait for the answer. Record in Pipedrive as `fthb_bss_outcome` (`shortlist_only`, `lender_intro_made`, or `signed_buyer_agreement`) within an hour of the call ending. If the prospect picked Option 2 or 3, set the calendar for the next step before ending the call.

> *"Thanks {{first_name}}. Shortlist PDF tomorrow. Math sheet link is in your inbox already. Talk soon."*

End the call. Do not extend past 30 minutes without explicit consent. If the conversation needs more time:

> *"We've got five more minutes. Want to keep going, or want me to send the rest in writing?"*

---

## Shortlist PDF template

Personalized per prospect. Built from the math sheet and the call notes. Sent within 24 hours of the call.

Detailed structure lives in [12-bss-math-and-shortlist.md](12-bss-math-and-shortlist.md) under "What goes into the Shortlist PDF." The voice and layout rules:

- Title: **Your Orlando Buyer Shortlist** in large type. Subtitle: prospect's name and date.
- Voice: agent's voice, second person, direct. Never refer to the prospect in the third person, never use "the buyer" or "the client."
- Math summary lives in a single box on the cover page so it survives a half-second skim.
- One page per ZIP. Same fields on every page (math, comp, trade-off, gotcha). Never skip a field; if a field is empty for a ZIP, that ZIP does not belong on the list.
- Closing page restates the three options from the call's close, the agent's contact, and a sentence about the math-sheet link being live and editable.

---

## Lender Comparison Card

A 1-page printable PDF the prospect can keep. Three lenders. Reusable across prospects but updated quarterly as lender capacity and rate-sheet competitiveness change.

### Template structure

Cover line:
> Three lenders I've personally worked with along the Sanford to Downtown Orlando corridor. Phone numbers below. None of them pay me a referral fee. Mention my name; you'll be expected.

For each lender, one row:

- **Name and company.**
- **Direct phone and email.** Not a switchboard line. Not "info@."
- **One-line "best for":** *"Best for credit in the 640 to 680 range."* *"Fastest close, regularly hits 21 days."* *"FHA and 3.5% down specialist; very patient with first-time buyers."*
- **What to ask for when you call:** *"Tell them I sent you for a buyer pre-approval. Ask for a Loan Estimate, not a quote. The Loan Estimate is the federal form; quotes are marketing."*
- **What to expect:** Typical response time for a first call. Typical timeline to a written pre-approval letter once docs are provided.

Footer line:
> I do not collect a referral fee from any of these lenders. You can call all three. Many prospects do.

---

## Red-Flag Property Filter

A 1-page printable PDF. Reusable across all prospects (not personalized). The agent updates the list as new corridor-specific patterns emerge.

### Voice line at the top

> Seven things first-time buyers in Orlando lose money on. Read this before every showing. If you can answer "no, we checked, it's fine" on all seven before the listing agent gets your offer, you've already filtered out 90% of the regret cases I see.

### The seven items

1. **Permits on additions.** A converted garage or enclosed lanai without a permit is not part of the legal square footage and will fail an appraisal. Ask the listing agent: "Were these additions permitted? Can I see the closed permits in the Seminole or Orange County records?" If the answer is vague, that's a no.

2. **Septic-to-sewer transition zones.** Some streets in older Sanford and Apopka are still on septic. Some are mid-transition with a mandatory hookup deadline. Ask the listing agent which the home is on and whether any sewer-conversion assessment is pending. A pending assessment can run $5,000 to $20,000 and the buyer inherits it.

3. **Roof age vs insurance carrier appetite.** Florida HO3 carriers will refuse to write a policy on a roof older than 15 years in many cases, or will charge a punitive premium. Ask the listing agent the year the roof was installed and request the wind-mitigation inspection report. If neither exists, plan on the cost of a 4-point inspection ($150 to $250) before going under contract.

4. **HOA delinquency rates.** Some 2000s-era HOA developments have delinquency rates over 15%. That can affect future special assessments and resale lender approvals. Request the HOA's most recent financial statement and the percentage of homeowners current on dues.

5. **Flood zone (AE vs X).** Some Sanford streets near the St. Johns River and some Apopka parcels are in flood zone AE. Flood insurance on an AE property can add $700 to $2,500/year on top of HO3. Look up the parcel on the FEMA flood map (msc.fema.gov) before falling in love with the house.

6. **Foundation cracks on 1960s to 1980s slab homes.** Older Orlando-corridor homes were built on slabs without modern moisture barriers. Hairline cracks are common and usually cosmetic. Cracks wider than a quarter-inch or with vertical displacement need a structural engineer's letter before going under contract. Don't accept the seller's word.

7. **School-zone resale math.** A home zoned to a desirable school resells 8 to 15% faster and at a meaningful premium. A home zoned to a weak elementary in an otherwise nice neighborhood is a quiet long-term loss. Even if you don't have kids, the zoning affects resale. Look up the parcel's specific elementary zone on the county school district map; do not trust the listing flyer.

---

## Math sheet shareable view

The math sheet's structure and formulas live in [12-bss-math-and-shortlist.md](12-bss-math-and-shortlist.md). The user-facing copy on the sheet:

- **Tab labels and cell labels:** plain English, second person where it works ("Your cash on hand," "Your target price," not "Cash available" or "Target purchase price"). Florida-specific items use Florida-specific names ("Doc stamps," not "Transfer tax"; "Save Our Homes cap," not "Property tax cap").
- **Cell tooltips on the input cells:** one line each, explaining what to type. *"Round numbers are fine."* *"This is the number you have available for closing day; leave the rest as savings."*
- **The summary box at the top of the Loan Scenarios tab:** **"Your all-in monthly,"** **"Your cash to close,"** **"Reserves after close."** These three labels are load-bearing. Do not change them to financial-industry shorthand.
- **The footer of every tab:**
  > Math sheet built for {{first_name}} on {{call_date}}. Link is yours to keep. Re-run anytime as your numbers or rates move.

---

## Voice and copy rules for the BSS surfaces

Inherited from [CLAUDE.md](../CLAUDE.md) and the "Voice and copy rules" section of [09-bss-offer-spec.md](09-bss-offer-spec.md). Specifically for this doc's surfaces:

- **Diagnostician, not salesperson.** Landing page shows what's *in* the call before it shows the booking button. The reader can predict the agenda from the page alone.
- **Either way works for me.** The Option 1 / Option 2 / Option 3 close is the spine. Mirror it in the FAQ, in the closing page of the Shortlist PDF, and in the post-call emails (see [11-bss-emails.md](11-bss-emails.md)).
- **Never confidently quote a number you have not validated.** Applies on the call, on the math sheet, in the Shortlist PDF, and in the Lender Comparison Card. If a number is stale, say so out loud.
- **No em dashes in body copy.** Period, comma, colon, or parens. Short separators in titles or table cells are fine.
- **No "corridor" in user-facing copy.** Use "the Sanford to Downtown Orlando route," "the Orlando metro," or name the specific ZIPs. "Downtown" alone is ambiguous; always say "Downtown Orlando."
- **Default property type is single-family.** Use townhomes when the math warrants it. Use condos only when the prospect explicitly asks or when the math forces it as the only fit.

---

## Open questions to resolve before locking

- **Lock the public subhead.** The proposed subhead at the top of this doc needs a Hormozi voice pass. Lock once and propagate verbatim to landing page, calendar tool description, every email subject, and the DM script.
- **Calendar tool selection.** Cal.com is the default per [08-implementation-roadmap.md](08-implementation-roadmap.md); switching is reversible but the booking-link structure needs to be locked before the landing page goes live.
- **The intake form delivery surface.** Native Astro form posting to Make.com is the default; Tally or Typeform is a credible alternative if the embed-in-Pipedrive-email path is easier. Pick before building.
- **Shortlist PDF authoring tool.** Notion-to-PDF, Google Docs template, or a typed Astro page rendered to PDF via headless Chrome. Pick during build. Whichever wins, the structure in this doc and in [12-bss-math-and-shortlist.md](12-bss-math-and-shortlist.md) is the template.
- **Red-flag filter localization.** The seven items here are corridor-specific to Orlando. If the agent ever opens a second metro, the filter needs to be re-localized; it should not be parameterized at build time.

---

## Related documents

- [09-bss-offer-spec.md](09-bss-offer-spec.md) — Offer, run-of-show framing, Pipedrive routing, hard rules
- [11-bss-emails.md](11-bss-emails.md) — Booking confirmation, reminders, deliverables, follow-up, no-show
- [12-bss-math-and-shortlist.md](12-bss-math-and-shortlist.md) — Math sheet structure, formulas, shortlist decision logic, own-zip handling
- [04-readiness-filter-emails.md](04-readiness-filter-emails.md) — Tier A nurture, the primary email surface that drives BSS bookings
- [08-implementation-roadmap.md](08-implementation-roadmap.md) — When each artifact ships
- [CLAUDE.md](../CLAUDE.md) — Funnel namespace, hard rules, market context
