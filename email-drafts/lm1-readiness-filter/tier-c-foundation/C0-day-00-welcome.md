---
id: LM1-C0
funnel: fthb
magnet: lm1
tier: C (FOUNDATION)
campaign: FTHB LM1 - Tier C
day: 0
trigger: ~10 minutes after transactional
goal: Reframe "Foundation Phase" as positive; set the cadence and retake hook
subject: What "Foundation Phase" actually means
status: DRAFT
last_edited: 2026-05-17
source_doc: docs/04-readiness-filter-emails.md
hard_rule: NEVER pitch the BSS to a Tier C contact
---

# C0 — Day 0 (welcome)

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

## Polish notes
<!-- Capture edits, alternates, decisions here. -->

- [ ] Make sure "0% sales pitch" promise is honored across the whole rotation
- [ ] Confirm retake link CTA passes through `?src=fthb_lm1_retake` for analytics

## Merge fields used
- `{{first_name}}`
- `{{fthb_retake_link}}`
- `{{agent_first_name}}`
