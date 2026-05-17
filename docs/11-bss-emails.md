## What this doc is

The full email sequence around the Buyer Strategy Session: the three pre-call touchpoints, the post-call deliverable email, the day-7 check-in, the not-ready-yet redirect, and the two no-show follow-ups. Subject lines and full bodies are drafts; they need a polish pass before they ship.

The Pipedrive Workflow Automations that fire these emails and the custom fields they read live in [09-bss-offer-spec.md](09-bss-offer-spec.md). Math, shortlist branching, and the Shortlist PDF structure live in [12-bss-math-and-shortlist.md](12-bss-math-and-shortlist.md).

Status: **DRAFT**. When polished, individual emails should be split into `email-drafts/bss/*.md` files mirroring the structure of `email-drafts/lm1-readiness-filter/`, with the frontmatter format used there.

---

## v0 MVP cut — which emails ship first

Per the v0 scope in [09-bss-offer-spec.md](09-bss-offer-spec.md), v0 ships **only one email** from this doc: **BSS-4 (post-call deliverable)**. The agent sends it manually after each call with the Shortlist PDF link attached.

The booking confirmation and the 1-hour reminder are handled by **Cal.com's native emails** in v0. They are not as warm as BSS-1 and BSS-3 drafted below, but they confirm the time and deliver the video link, which is what the prospect needs. Re-author BSS-1 and BSS-3 in the agent's voice in v1, once the Pipedrive Campaign infrastructure exists.

| Email | v0 | v1 |
|---|---|---|
| BSS-1 (booking confirmation) | Cal.com default | This doc's draft |
| BSS-2 (24h reminder) | Skip | This doc's draft |
| BSS-3 (1h reminder) | Cal.com default | This doc's draft |
| BSS-4 (deliverable) | This doc's draft, sent manually | This doc's draft, fired by Workflow Automation |
| BSS-5 (day 7 check-in) | Skip | This doc's draft |
| BSS-6 (not ready yet) | Skip; agent emails a 1-paragraph custom note when this comes up | This doc's draft |
| BSS-NS-1 (no-show 1h) | Skip; agent emails the reschedule link manually if they want the prospect back | This doc's draft |
| BSS-NS-2 (no-show 7d) | Skip | This doc's draft |

In v0, **BSS-4 collapses to the `shortlist_only` body only**. The `lender_intro_made` variant is deferred along with the `lender_intro_made` outcome enum value (see [09-bss-offer-spec.md](09-bss-offer-spec.md)). If a v0 prospect picks Option 2 on the call, the agent sends the `shortlist_only` BSS-4 and a separate plaintext lender-intro email.

The full sequence below is the v1 design. v0 ships one email from it.

---

## Email index

| ID | When | Subject (proposed) | Goal |
|---|---|---|---|
| **BSS-1** | Immediately on booking | You're booked. Here's the 3-minute prep. | Confirm, deliver intake form, set expectations |
| **BSS-2** | T minus 24 hours | Tomorrow at {{time}}. One quick thing. | Reminder, video link, intake form nudge if blank |
| **BSS-3** | T minus 1 hour | In an hour. See you on the video link. | Final reminder, video link only |
| **BSS-4** | Within 24 hours of call | Your shortlist and your numbers, as promised. | Deliver Shortlist PDF, recap, restate three options |
| **BSS-5** | Day 7 after BSS-4 | Quick check-in. Anything come up? | Open the door for a question, no pitch |
| **BSS-6** | When `fthb_bss_outcome = not_ready_yet` | Reset. Here's where I'd actually start. | Frame the re-routing as a favor, restart nurture |
| **BSS-NS-1** | 1 hour after a no-show | Missed you. Want to pick another time? | Reschedule link, no guilt |
| **BSS-NS-2** | 7 days after BSS-NS-1 if no rebooking | Last one from me on the strategy session. | Sign-off, open door, no pressure |

---

## Merge fields used across this sequence

Set by Make.com or by Pipedrive Workflow Automations off the custom fields in [09-bss-offer-spec.md](09-bss-offer-spec.md). All field names use the `fthb_` namespace per [CLAUDE.md](../CLAUDE.md).

| Merge field | Source |
|---|---|
| `{{first_name}}` | Pipedrive Person |
| `{{agent_first_name}}` | Static (the agent) |
| `{{agent_last_name}}` | Static |
| `{{agent_license_no}}` | Static |
| `{{brokerage}}` | Static |
| `{{scheduled_time}}` | `fthb_bss_scheduled_for` |
| `{{scheduled_time_short}}` | `fthb_bss_scheduled_for` formatted as e.g. "Friday at 2:30 PM" |
| `{{video_link}}` | Cal.com webhook payload |
| `{{intake_form_link}}` | Static URL with `?email=` prefilled |
| `{{reschedule_link}}` | Cal.com per-contact reschedule URL |
| `{{shortlist_pdf_link}}` | Set by the agent when the PDF is delivered; stored on the Pipedrive Person |
| `{{math_sheet_link}}` | Set by the agent at booking; same link given live on the call |
| `{{recommended_lender_first_name}}` | Set by the agent when `fthb_bss_outcome = lender_intro_made` |
| `{{recommended_lender_company}}` | Same |
| `{{recap_one_line}}` | Set by the agent in the BSS-4 send (a single line of personalization referencing the call) |
| `{{redirect_tier_label}}` | Used by BSS-6; "90-Day Sprint" or "Foundation Phase" |
| `{{redirect_next_step_one_line}}` | Used by BSS-6; what tier-appropriate nurture starts |

Anywhere a body references a specific person on the call (co-buyer, lender, etc.), that name is also a merge field. Treat the call notes as the source of truth and store the few free-text references the email needs as Pipedrive note fields the agent fills in before triggering the email.

---

## BSS-1: Booking confirmation (Immediately on booking)

**Trigger:** Pipedrive Workflow Automation **BSS Booking Confirmation** fires when `fthb_bss_booked_at` is first set.

**Subject:** You're booked for {{scheduled_time_short}}. Here's the 3-minute prep.

**Preview text:** Calendar invite is on its way. One short form, optional. Then I'll see you on the video link.

**Body:**

```
Hey {{first_name}},

You're on my calendar for {{scheduled_time}}.

Calendar invite with the video link is on its way to this
email address. It'll be in your inbox within a minute or
two of this note. If it doesn't land, check spam and then
hit reply and I'll resend.

One ask before we talk. There's a 3-minute intake form
here:

    {{intake_form_link}}

Six questions, none of them require a credit score or any
documents. The more I know going in, the less of our 30
minutes we spend on warm-up. If you'd rather skip it and
do the warm-up live, that's fine. The form is a head start,
not a gate.

What to expect on the call:

1. I've already read your scorecard answers. I'll start by
   reading them back to you so you know I'm not going to ask
   you to re-explain anything.

2. We open a Google Sheet together. Your numbers, today's
   rate, the Florida tax and insurance lines most calculators
   miss. The link is yours after the call.

3. We look at three neighborhoods. If you have ones in mind
   already, those lead the list. If you don't, I'll bring
   three I'd pick.

4. Last three minutes, three options for what happens next.
   Pick whichever fits. No pressure, no paperwork.

Talk soon,
{{agent_first_name}}
{{agent_license_no}} | {{brokerage}}

P.S. If something comes up and you need to move the time,
the calendar invite has a reschedule link. Use it without
guilt. Better to move it than to take a call you're not
present for.
```

**Polish notes:**
- Confirm the intake form URL prefills the email so the prospect doesn't retype it.
- Confirm calendar invite is sent separately (Cal.com handles this; the email is for the human framing).
- Verify the P.S. doesn't read as inviting reschedules; it should read as removing a guilt barrier if one already exists.

---

## BSS-2: T minus 24 hours (24h before call)

**Trigger:** Pipedrive scheduled send, 24 hours before `fthb_bss_scheduled_for`.

**Subject:** Tomorrow at {{scheduled_time_short}}. One quick thing.

**Preview text:** Same video link, same 30 minutes. Quick nudge on the intake form if you didn't get to it.

**Body (when `fthb_bss_intake_completed_at` is null):**

```
Hey {{first_name}},

Quick reminder. We're on tomorrow at {{scheduled_time_short}}.

Video link is in your calendar invite. Same one as before.

I noticed the intake form didn't come back. Totally fine,
but if you have a couple of minutes today, the three most
useful answers for me are:

1. Are you bringing a partner or co-buyer? (Yes or no.)
2. Roughly what's your timeline? (Buying in 3 months, 6
   months, 12 months, or just exploring.)
3. Any neighborhoods you're already looking at? (Or do you
   want me to bring three based on your numbers?)

You can hit reply to this email with the three answers. Or
fill out the form:

    {{intake_form_link}}

Either way, see you tomorrow.

{{agent_first_name}}
```

**Body (when `fthb_bss_intake_completed_at` is set):**

```
Hey {{first_name}},

Quick reminder. We're on tomorrow at {{scheduled_time_short}}.

Got your form. Read it twice. Already pre-loaded the math
sheet with your numbers so we can move fast.

Video link is in your calendar invite.

If a partner or co-buyer is coming, make sure they're on
the call too. The math we do only works for both of you
if both of you watch it happen live.

Talk tomorrow,
{{agent_first_name}}
```

**Polish notes:**
- Confirm the conditional branch is set up in Pipedrive Workflow Automation against `fthb_bss_intake_completed_at`.
- "Read it twice" is load-bearing for perceived likelihood; do not soften.

---

## BSS-3: T minus 1 hour (1h before call)

**Trigger:** Pipedrive scheduled send, 1 hour before `fthb_bss_scheduled_for`.

**Subject:** In an hour. See you on the video link.

**Preview text:** Video link below. Bring your co-buyer if you have one. That's it.

**Body:**

```
Hey {{first_name}},

In an hour. Video link:

    {{video_link}}

If anything blew up and you need to reschedule, here:

    {{reschedule_link}}

Otherwise, see you in an hour.

{{agent_first_name}}
```

**Polish notes:**
- Keep it short. T-1h emails that try to do more get skimmed past the link.
- Reschedule link is in here on purpose; the cost of a no-show is higher than the cost of a same-day reschedule.

---

## BSS-4: Post-call deliverables (within 24 hours of call)

**Trigger:** Pipedrive Workflow Automation **Post-call Routing** fires when `fthb_bss_outcome` is set to `shortlist_only` or `lender_intro_made`. Agent attaches the Shortlist PDF link before triggering.

**Subject:** Your shortlist and your numbers, as promised.

**Preview text:** PDF below. Math sheet link is the same one from the call. Three options, as discussed.

**Body (when `fthb_bss_outcome = shortlist_only`):**

```
Hey {{first_name}},

Shortlist PDF:

    {{shortlist_pdf_link}}

Math sheet (same link from the call, still live, still
editable):

    {{math_sheet_link}}

{{recap_one_line}}

Three things in the PDF:

1. Your buy-box in one box on the cover. Target price,
   monthly all-in, cash to close, reserves after close.

2. One page per neighborhood, with a real comp on each
   page from yesterday's MLS pull. The trade-off and one
   gotcha specific to that ZIP are below the comp.

3. The three options we talked about on the last page.
   Shortlist only, lender intro, buyer's agent. You can
   pick a different one next week than the one you picked
   on the call. That's fine.

I'm not going to follow up to push for a decision. The
shortlist is yours either way. If a question comes up
later, hit reply and I'll answer it.

{{agent_first_name}}
{{agent_license_no}} | {{brokerage}}

P.S. The math sheet is the highest-leverage thing I sent
you. Open it the day before you talk to any lender, plug
in their quoted rate, and check whether the all-in monthly
they're showing you matches the all-in monthly you see in
the sheet. If it doesn't, ask them to itemize.
```

**Body (when `fthb_bss_outcome = lender_intro_made`):**

```
Hey {{first_name}},

Shortlist PDF:

    {{shortlist_pdf_link}}

Math sheet (same link from the call):

    {{math_sheet_link}}

{{recap_one_line}}

On the lender intro: I'm sending {{recommended_lender_first_name}}
at {{recommended_lender_company}} a separate email today
with you copied. They'll reach out to schedule a 20-minute
pre-approval call. No pressure to use them; you can call
all three lenders from the comparison card if you want a
second opinion. Plenty of people do.

When you talk to them, ask for a Loan Estimate, not a
quote. The Loan Estimate is the federal form that has to
itemize everything. Quotes are marketing.

After the pre-approval call, you'll have a number. That's
the number the shortlist gets re-run against if it moves
much from the math sheet.

{{agent_first_name}}
{{agent_license_no}} | {{brokerage}}

P.S. If anything on the pre-approval call feels off, hit
reply. Reading lender pitches is half my job.
```

**Polish notes:**
- `{{recap_one_line}}` is one sentence the agent writes per prospect referencing something specific from the call. Examples: *"That math on the FHA vs conventional gap surprised both of us; the conventional 5% scenario is the one you said felt safest."* or *"Confirming Sanford is your anchor and we added Casselberry as the wildcard."* No template here on purpose; it's the personalization that makes the email feel hand-written.
- Confirm the Shortlist PDF link is a stable URL (not a one-time download) so the prospect can re-open it months later.
- The P.S. on the lender-intro version doubles as a soft promise that the agent will weigh in on lender quality if asked, without volunteering it.

---

## BSS-5: Day 7 check-in

**Trigger:** Pipedrive scheduled send, 7 days after BSS-4 sends, for prospects with `fthb_bss_outcome` in (`shortlist_only`, `lender_intro_made`).

**Subject:** Quick check-in. Anything come up?

**Preview text:** A week since our call. If a question's been bouncing around, this is your reply window.

**Body:**

```
Hey {{first_name}},

A week since we talked. Three things, then I'll get out of
your inbox.

1. The shortlist PDF is still where I sent it. If you've
   read through it and any of the three neighborhoods looks
   different now than it did on the call, hit reply.

2. The math sheet is still live. If you want me to plug in
   a different rate or a different cash number to see what
   moves, send me the numbers and I'll re-run it. Takes me
   ten minutes.

3. If you've gotten a pre-approval (from the lender I
   introduced or from anyone else), and the number is more
   than $20,000 off what the math sheet showed, that's
   worth a five-minute follow-up call. Otherwise, no need.

Either way, I'll be sending you the monthly Orlando market
update starting around two weeks from now. You can
unsubscribe from that anytime without unsubscribing from me.

{{agent_first_name}}

P.S. If nothing's come up and you're sitting with the
shortlist, that's the right move. Big decisions don't
need to be fast.
```

**Polish notes:**
- The three-item list is the spine. Each item is a different ramp back into a conversation; the prospect picks whichever feels lightest.
- The P.S. is doing emotional work. First-time buyers who feel pressured stop reading. The P.S. removes the pressure that "no movement = something is wrong."

---

## BSS-6: Not-ready-yet redirect

**Trigger:** Pipedrive Workflow Automation **Post-call Routing** fires when `fthb_bss_outcome = not_ready_yet`. The agent has already updated `fthb_lm1_tier` to the appropriate downgrade (B or C).

**Subject:** Reset. Here's where I'd actually start.

**Preview text:** Honest take after our call. You're not behind. You're at a different step than the tier letter suggested.

**Body:**

```
Hey {{first_name}},

Thanks for being honest on the call. Two things came out
that the scorecard didn't catch:

{{recap_one_line}}

Based on that, you're not in the same spot as the score
suggested. You're closer to {{redirect_tier_label}}.

That's not a downgrade. It's a different starting point
with a different next step.

{{redirect_next_step_one_line}}

Concretely, here's what I'm doing on my end:

1. I'm enrolling you in the {{redirect_tier_label}} email
   sequence instead. Lighter cadence, more education, less
   "ready to buy" framing. You can unsubscribe anytime.

2. I'm keeping your math sheet live so you can come back
   to it as your numbers change. Same link as before:

       {{math_sheet_link}}

3. I'm not going to book another strategy session with
   you for at least 90 days. The session is for buyers
   who are ready to make a decision in the next 90 days,
   not for everyone. When you're closer, we book again.
   You'll know.

If you want to talk in the meantime about anything that's
not "should I buy a house right now," I read every reply.

{{agent_first_name}}
{{agent_license_no}} | {{brokerage}}

P.S. Most of the people who tell me "actually I'm not as
ready as I thought" end up buying within 18 months. The
ones who pretend they're ready when they're not are the
ones who lose money. You did the right thing.
```

**Polish notes:**
- The P.S. is critical and load-bearing. It reframes the redirect from rejection to validation. Do not cut.
- Confirm the redirect re-enrollment in Pipedrive happens before this email sends; otherwise the prospect gets confused about which sequence they are on.
- `{{recap_one_line}}` here is specifically about what surfaced on the call that changed the tier. Examples: *"The down payment is from a 401k loan that hasn't funded yet, and the timing on it is closer to 8 months than 3."* The honesty is the deal.

---

## BSS-NS-1: First no-show follow-up (1 hour after missed slot)

**Trigger:** Pipedrive Workflow Automation **No-Show Handling** fires when `fthb_bss_scheduled_for < now()` AND `fthb_bss_attended_at` is null.

**Subject:** Missed you. Want to pick another time?

**Preview text:** Reschedule link below. No guilt, no follow-up if you'd rather not.

**Body:**

```
Hey {{first_name}},

Looks like we missed each other at {{scheduled_time_short}}.

Two doors:

Door 1. Pick a different time:

    {{reschedule_link}}

Door 2. Hit reply and tell me you'd rather not do this
right now. I'll close the loop and you won't hear from me
on this again.

Either is fine. I don't take no-shows personally; life
happens.

{{agent_first_name}}
```

**Polish notes:**
- Length is the point. A long no-show recovery email reads as guilt-tripping.
- "Two doors" mirrors the "Option 1 / Option 2 / Option 3" spine elsewhere in the sequence, scaled down. Same diagnostician posture.

---

## BSS-NS-2: Final no-show follow-up (7 days after BSS-NS-1)

**Trigger:** Pipedrive Workflow Automation fires 7 days after BSS-NS-1 if no rebooking has happened and the prospect has not opted out.

**Subject:** Last one from me on the strategy session.

**Preview text:** Door's open whenever. After this you won't see another reminder.

**Body:**

```
Hey {{first_name}},

Last note on the strategy session. After this I'll stop
sending these.

If life got loud and you want to find another time, the
link still works:

    {{reschedule_link}}

If the moment's passed for now, that's fine too. You'll
keep getting the monthly Orlando market update unless you
unsubscribe. The strategy session is open to you whenever
you want it; I don't take it off the table.

Either way, no follow-up from me on this after today.

{{agent_first_name}}
{{agent_license_no}} | {{brokerage}}

P.S. The reason I cap it at two emails is that anyone who
needs three reminders to book a free 30-minute call wasn't
going to find the call useful when they got on it anyway.
Better for both of us.
```

**Polish notes:**
- The P.S. is the only place this sequence breaks the diagnostician voice slightly; it explains the cadence. That explanation is load-bearing because the user is letting them off the hook *and* explaining why. Without the P.S. the cap looks arbitrary.
- After this email, the prospect returns to whatever Campaign they were paused from (LM1 Tier A nurture, LM2 nurture, or monthly market update only).

---

## Sequence-wide rules

These apply to every email in this doc and to the future Spanish-language versions once those ship.

1. **Single CTA per email.** The CTA is either the intake form, the video link, the reschedule link, the math sheet link, the Shortlist PDF link, or the reply. Never two CTAs in the same email. The exception is BSS-4, where the Shortlist PDF and the math sheet are bundled because they are one deliverable in two parts.

2. **The agent reads every reply.** This is stated in BSS-1 and lived everywhere else. Replies are routed to the agent's inbox, not an auto-responder. If the agent cannot honor this, the line gets cut from BSS-1 first.

3. **No "tap to schedule" or "click here" link copy.** Every link gets a short label or a literal URL on its own indented line. Bare URLs read as honest in plain-text email and survive being forwarded.

4. **No em dashes in body copy.** Period, comma, colon, or parens. Em dashes are reserved for short separators in subject lines or labels.

5. **No "Don't miss out" or scarcity framing.** Anywhere. Per [CLAUDE.md](../CLAUDE.md), the project rule from $100M Offers is *"making the customer feel like they're getting a deal, not being sold to."* Scarcity reads as a sale.

6. **No track-record references.** The agent is newly licensed (per memory). Filter for process, skill, and honesty. Never reference deal count, years of experience, or "I've helped X buyers."

7. **No "we" when "I" is honest.** The agent is solo. "We" makes the brokerage feel like a team it isn't. "I" is the voice; "I and the lender I'm introducing you to" is fine when it's true.

8. **Numbers in any email are validated before they ship.** Per memory: *"validate numbers in copy before shipping."* Rates, payment estimates, insurance ranges, anything quantitative. If a number is in an email, it has been run.

---

## Open questions to resolve before locking

- **Subject line A/B testing.** The proposed subjects need to be tested against open rate, especially BSS-4 and BSS-5 (the post-call retention emails). Until there's data, the proposed subjects are the defaults.
- **BSS-5 cadence.** Day 7 is a guess. If the agent observes that day 3 or day 10 lands better, calibrate in [09-bss-offer-spec.md](09-bss-offer-spec.md)'s config sketch.
- **BSS-6 redirect emotional pitch.** The "P.S. Most of the people who tell me 'actually I'm not as ready as I thought' end up buying within 18 months" line is doing a lot of emotional work and is asserting a claim about behavior. The agent should confirm this is true in their own pipeline before letting the line ship; if not, rewrite the P.S. without the claim.
- **The "monthly Orlando market update" referenced in BSS-5 and BSS-NS-2.** This is the **FTHB Monthly Market Update** Campaign from the funnel namespace in [CLAUDE.md](../CLAUDE.md). It needs to exist before either of these emails reference it. If it does not exist yet, cut the reference from both emails until it ships.
- **Splitting into draft files.** When this doc moves out of DRAFT, each email should get its own file in `email-drafts/bss/` with the same frontmatter format used in `email-drafts/lm1-readiness-filter/`.

---

## Related documents

- [09-bss-offer-spec.md](09-bss-offer-spec.md) — Offer, run-of-show, Pipedrive Workflow Automations that fire these emails, custom fields
- [10-bss-content.md](10-bss-content.md) — Landing page, intake form, in-call script, deliverable templates
- [12-bss-math-and-shortlist.md](12-bss-math-and-shortlist.md) — Math, formulas, shortlist decision logic, own-zip handling
- [04-readiness-filter-emails.md](04-readiness-filter-emails.md) — The voice template for nurture sequences in this project
- [CLAUDE.md](../CLAUDE.md) — Voice rules, hard rules, market context, namespace
