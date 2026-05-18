---
id: LM1-A3
funnel: fthb
magnet: lm1
tier: A (READY_NOW)
campaign: FTHB Readiness Quiz - Ready Now Day 5
day: 5
trigger: 5 days after A1
goal: Triage their letter; create a low-friction reply hook
subject: The pre-approval question I always ask first
status: DRAFT
last_edited: 2026-05-17
source_doc: docs/04-readiness-filter-emails.md
---

# A3 — Day 5

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

## Polish notes
<!-- Capture edits, alternates, decisions here. -->

- [ ] Consider whether "redacted copy" line needs an inline definition for first-timers
- [ ] Confirm the two CTAs (reply + book) don't dilute each other

## Merge fields used
- `{{first_name}}`
- `{{book_bss_link}}`
- `{{agent_first_name}}`
