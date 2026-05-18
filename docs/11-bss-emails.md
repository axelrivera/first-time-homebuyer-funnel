## What this doc is

The optional plaintext follow-up note the agent may send after a BSS that ended in `not_yet`. **There is no automated email sequence around the BSS.** Google Calendar handles the booking confirmation and the reminder email natively; everything else is at the agent's discretion.

This is by design. The BSS is a closing conversation in a funnel where LM1 and LM2 already did the value-delivery work (see [09-bss-offer-spec.md](09-bss-offer-spec.md)). Stacking a structured post-call email sequence on top reintroduces the value-stack framing the BSS is deliberately stepping away from.

Status: **DRAFT**. The follow-up template is a starting point; the agent personalizes per call.

---

## When to send

After a BSS that ended in `not_yet`, the agent decides whether to send a follow-up. Send it when:

- The conversation surfaced a specific question or number the agent committed to confirming.
- The prospect asked for the agent to do something concrete (e.g., "send me the lender's contact info").
- The agent wants to leave one clean door for re-engagement before the prospect returns to nurture.

Do not send when:

- The prospect said clearly they were not ready and did not need more from the agent. Respect that.
- The conversation was warm but the next concrete touchpoint is the lender introduction itself. Send the lender intro email instead; it does double duty.

If the prospect signed the BBA on the call (`signed`), the active-client onboarding flow takes over and this email is not used.

If the prospect no-showed (`no_show`), the agent decides whether to email the reschedule link or let it close. No template here; that is a two-line note in the agent's voice.

---

## The optional follow-up template

Send within 24 hours of the call. Plaintext. Short.

**Subject:** Quick note after our call

**Body:**

```
Hey {{first_name}},

Thanks for the time today. Quick recap of where we landed:

{{one_line_recap}}

{{next_step_specific_to_this_prospect}}

No follow-up from me unless you want one. The door's open
whenever you want to revisit.

{{agent_first_name}}
{{agent_license_no}} | {{brokerage}}
```

### Merge field guidance

- **`{{one_line_recap}}`**: a single sentence in the agent's voice referencing something specific from the call. *"You're targeting March, cash is solid at the $30k range, and Sanford and Lake Mary are the two neighborhoods you're focused on."* Not a transcript; a confirmation that the agent listened.
- **`{{next_step_specific_to_this_prospect}}`**: one paragraph naming whatever the agent committed to or whatever the prospect committed to. Examples:
  - *"I'll email {{lender_first_name}} at {{lender_company}} tomorrow with you cc'd. They'll reach out to schedule a 20-minute pre-approval call."*
  - *"Let's revisit in 30 days once you've talked to your dad about the gift letter."*
  - *"Nothing on my side. The ball's in your court whenever you're ready to move."*

### What this email does not do

- It does not pitch the BSS or any other offer. The BSS already happened.
- It does not include a calendar link to "book another time."
- It does not stack bonuses or remind the prospect of value.
- It does not run on a schedule. The agent sends it once, manually, within 24 hours of the call.

---

## Voice and copy rules for the follow-up

Inherited from [CLAUDE.md](../CLAUDE.md):

- **Short.** Three short paragraphs maximum. No P.S. unless one specific call demands it.
- **No em dashes in body copy.** Period, comma, colon, or parens.
- **"I" not "we."** The agent is solo.
- **No "looking forward to" or other filler closes.** The closing line is functional ("door's open whenever") not ornamental.
- **No track-record references.** The agent is newly licensed.
- **Validate any numbers before they ship.** If a payment or rate ends up in the recap, run the math; do not approximate.

---

## What used to live here

Earlier drafts of this doc carried an eight-email sequence around the BSS (BSS-1 booking confirmation, BSS-2 24h reminder, BSS-3 1h reminder, BSS-4 deliverable, BSS-5 day-7 check-in, BSS-6 not-ready-yet redirect, BSS-NS-1 and BSS-NS-2 no-show follow-ups). That sequence has been removed. The reasoning:

- Google Calendar's native confirmation and reminder cover the pre-call touchpoints.
- The BSS no longer sends a personalized post-call deliverable (no Shortlist PDF, no lender card), so the "deliverable email" has nothing to deliver.
- The "day-7 check-in" and "not-ready-yet redirect" were value-stack moves that re-pitch the offer after the call. The reframed BSS is a closing conversation; the answer was on the call.
- No-show follow-ups are a manual judgment per prospect, not an automated cadence.

If volume grows and the data justifies it (e.g., a `not_yet` cohort that consistently re-engages weeks later, or a no-show rate above 15%), revisit the sequence then. Until then, the optional follow-up above is the entire email surface around the BSS.

---

## Related documents

- [09-bss-offer-spec.md](09-bss-offer-spec.md) — Offer, outcome recording, hard rules
- [10-bss-content.md](10-bss-content.md) — In-call structure
- [12-bss-math-and-shortlist.md](12-bss-math-and-shortlist.md) — The agent's internal worksheet
- [CLAUDE.md](../CLAUDE.md) — Voice, hard rules
