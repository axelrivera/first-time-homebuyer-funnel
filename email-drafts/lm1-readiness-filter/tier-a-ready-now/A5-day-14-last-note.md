---
id: LM1-A5
funnel: fthb
magnet: lm1
tier: A (READY_NOW)
campaign: FTHB Readiness Quiz - Ready Now Day 14
day: 14
trigger: 14 days after A1
goal: Soft sign-off, keep door open, referral hook (P.S.)
subject: Last note from me
status: DRAFT
last_edited: 2026-05-17
source_doc: docs/04-readiness-filter-emails.md
post_sequence_routing: After A5 → monthly market-update list
---

# A5 — Day 14

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

## Polish notes
<!-- Capture edits, alternates, decisions here. -->

- [ ] Confirm referral-loop P.S. matches the locked LM1 title language
- [ ] Verify the Pipedrive Workflow Automation actually moves them to the monthly list after this send

## Merge fields used
- `{{first_name}}`
- `{{book_bss_link}}`
- `{{agent_first_name}}`
- `{{fthb_readiness_quiz_link}}`
