---
id: LM1-B1
funnel: fthb
magnet: lm1
tier: B (NINETY_DAY)
campaign: FTHB Readiness Quiz - 90-Day Sprint Day 0
day: 0
trigger: ~10 minutes after transactional
goal: Deliver/re-deliver LM2 (the 9-Step Roadmap)
subject: "Your 90-day game plan: start here"
status: DRAFT
last_edited: 2026-05-17
source_doc: docs/04-readiness-filter-emails.md
---

# B1 — Day 0

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

## Polish notes
<!-- Capture edits, alternates, decisions here. -->

- [ ] Verify the three "you won't find" bullets stay accurate to the LM2 content
- [ ] Decide whether to mirror locked LM2 subhead language ("…between 'I think I'm ready' and keys in your hand")

## Merge fields used
- `{{first_name}}`
- `{{fthb_lm2_optin_link}}`
- `{{agent_first_name}}`
