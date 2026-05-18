## What this doc is

The shipping copy and structure for the **Buyer Strategy Session (BSS)** call itself. The high-level offer (eligibility, outcome recording, hard rules) lives in [09-bss-offer-spec.md](09-bss-offer-spec.md). The agent's internal worksheet math lives in [12-bss-math-and-shortlist.md](12-bss-math-and-shortlist.md). The optional follow-up email template lives in [11-bss-emails.md](11-bss-emails.md).

Status: **DRAFT**. The in-call structure is the load-bearing artifact in this doc. The CTA copy fragments at the end propagate to the funnel surfaces that link to the booking page.

---

## Booking surface

There is no BSS landing page. The booking surface is the agent's existing **Google Calendar appointment scheduling page**, using its standard UI. The agent does not customize copy on it beyond:

- Appointment title: **Buyer Strategy Session (30 min)**
- Description: *"A 30-minute video call to talk through your specific situation: timeline, cash, neighborhoods, lender questions. We'll figure out the right next step for you together."*

Google Calendar handles the booking confirmation and the reminder email natively.

---

## CTA copy (for the surfaces that link to the booking page)

The button label is the same on every surface. The supporting line above it changes by surface (LM1 Tier A result page, A1 through A5 emails, LM2 viewer page, LM2 emails). Owning docs: [03-readiness-filter-content.md](03-readiness-filter-content.md), [04-readiness-filter-emails.md](04-readiness-filter-emails.md), [06-process-map-content.md](06-process-map-content.md), [07-process-map-emails.md](07-process-map-emails.md).

**Button label (locked across all surfaces):**

> Book a 30-Minute Call with {{agent_first_name}}

**Above-the-button line on the LM1 Tier A result page:**

> You're ready. Let's talk through your specific situation and figure out the right next step.

**Above-the-button line in Tier A and LM2 nurture emails:**

Varies by email position. See [04-readiness-filter-emails.md](04-readiness-filter-emails.md) and [07-process-map-emails.md](07-process-map-emails.md) for the per-email copy. The pattern in all cases: one sentence of context, then the button. No bulleted value stack, no guarantee callout, no promised deliverables.

---

## In-call structure

Three blocks, directional not scripted. The agent walks in with the prospect's Pipedrive record open and the worksheet from [12-bss-math-and-shortlist.md](12-bss-math-and-shortlist.md) on a second tab if useful.

### Block 1: Recap and reset (0:00 to 5:00)

Agent reads back what they already know from LM1 (and LM2, if applicable) in one or two sentences.

Sample frame (not a script; rephrase in voice):

> *"Hey {{first_name}}, thanks for booking. From your scorecard I know you're targeting {{timeline}}, you came back as Tier {{tier_letter}}, and your cash is roughly in the {{cash_bucket}} range. What's changed since you took the quiz?"*

The "what's changed" question is the most important sentence in this block. It surfaces anything that has moved (rate, job, partner conversation) without making the prospect re-explain their starting state.

### Block 2: Diagnose and answer (5:00 to 20:00)

The agent runs through the diagnostic angles in whatever order the conversation pulls them:

- **Timeline.** Confirm or correct. Anchor the rest of the conversation against it.
- **Cash.** Down payment available, source (savings, gift, 401k, home sale). Reserves expectation.
- **Neighborhoods.** Where they are looking, what's drawing them there, what they have ruled out.
- **Lender status.** Pre-approved, talking to one, none. If none, this is the highest-leverage gap to flag.
- **Deal-breakers.** What they will walk away from. Not the wish list; the hard line.
- **Backup plan.** If they don't buy in the next six months, what happens?

The agent also answers the specific questions the prospect brought (most bring one or two). If a number question comes up (payment, closing costs, insurance ballpark), the agent uses the worksheet in [12-bss-math-and-shortlist.md](12-bss-math-and-shortlist.md) to compute it on their own screen. The worksheet is not shared by default; if the agent chooses to screen-share to answer a specific number, the goal is to answer and move on, not to make the worksheet part of the offer.

### Block 3: Next step (20:00 to 30:00)

The agent names three options and asks for a decision:

> *"Three options for what happens next. Option 1: we sign the buyer brokerage agreement today and start the search. Option 2: I introduce you to a lender this week, you take a pre-approval call, and we talk again once you have a number. Option 3: we stay in touch through email, you keep getting the monthly market update, and we revisit in 30 to 60 days. Which one fits?"*

The agent does not soften past the question. The 10 minutes here are for talking through the choice, not for filibustering.

After the answer, the agent records the outcome in Pipedrive within 1 hour of the call ending (`signed`, `not_yet`, or `no_show`).

End at 30 minutes. If the prospect wants more time:

> *"We've got five more minutes. Want to keep going, or want me to send the rest in writing?"*

---

## Voice and copy rules for the BSS surfaces

Inherited from [CLAUDE.md](../CLAUDE.md) and the voice rules in [09-bss-offer-spec.md](09-bss-offer-spec.md). Specifically for this doc's surfaces:

- **No promises of deliverables.** No personalized shortlist, no lender comparison card, no math sheet share link, no red-flag filter. The funnel already delivered value; the call is the conversation.
- **Short CTAs, no stacked bullet lists of value.** Anywhere the BSS is mentioned in a funnel email or page, it is one sentence plus a button.
- **"Book a call," not "Book your strategy session" or "Reserve your spot."** The button label above is locked.
- **At the close on the call, ask for the decision.** Don't ramp into a soft "let me know what you think." The decision is the point of the call.
- **No em dashes in body copy.** Period, comma, colon, or parens.
- **No "we" when "I" is honest.** The agent is solo.
- **No track-record references.** The agent is newly licensed; filter for process and honesty.

---

## Open questions to resolve before locking

- **Button label test.** "Book a 30-Minute Call with {{agent_first_name}}" is a starting point. If real Tier A click-through rates underperform, test variants ("Talk to {{agent_first_name}}", "30 Minutes with {{agent_first_name}}"). Lock after the first cohort of Tier A scorecards has landed.
- **Google Calendar appointment description.** The one-sentence description above is a starting point. Adjust after the agent has run five BSSes and knows what the prospect expects walking in.
- **Whether Block 3 names the BBA on the call** or stays softer ("we sign a representation agreement"). The full Florida term (Buyer Brokerage Agreement) is the correct one per project memory. Decide whether to use the term verbally or only on paperwork.

---

## What used to live here

Earlier drafts of this doc carried a full public landing page (hero, four-bonus value stack, 30-Minute Promise guarantee, FAQ), a six-question intake form, a five-block scripted run-of-show, a Shortlist PDF template, a Lender Comparison Card PDF, and a Red-Flag Property Filter PDF. Those have been removed. The reasoning is in [09-bss-offer-spec.md](09-bss-offer-spec.md): the BSS is the conversion conversation, not a stand-alone Grand Slam Offer with its own deliverables. The funnel (LM1, LM2) is where value is delivered.

---

## Related documents

- [09-bss-offer-spec.md](09-bss-offer-spec.md) — Offer, eligibility, outcome recording, hard rules
- [11-bss-emails.md](11-bss-emails.md) — The single optional follow-up template
- [12-bss-math-and-shortlist.md](12-bss-math-and-shortlist.md) — The agent's internal worksheet
- [03-readiness-filter-content.md](03-readiness-filter-content.md) — Tier A result page CTA
- [04-readiness-filter-emails.md](04-readiness-filter-emails.md) — Tier A emails A1 through A5 CTAs
- [06-process-map-content.md](06-process-map-content.md) — LM2 viewer page CTA
- [07-process-map-emails.md](07-process-map-emails.md) — LM2 nurture CTAs
- [CLAUDE.md](../CLAUDE.md) — Voice, hard rules, market context
