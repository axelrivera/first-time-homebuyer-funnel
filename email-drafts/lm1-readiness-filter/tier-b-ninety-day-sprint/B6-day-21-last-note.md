---
id: LM1-B6
funnel: fthb
magnet: lm1
tier: B (NINETY_DAY)
campaign: FTHB LM1 - Tier B
day: 21
trigger: 21 days after B1
goal: Soft sign-off; hand off to monthly market update
subject: "Last note (then I back off)"
status: DRAFT
last_edited: 2026-05-17
source_doc: docs/04-readiness-filter-emails.md
post_sequence_routing: After B6 → monthly market-update list (unless `fthb_received_lm2 = true`, in which case Pipedrive Workflow Automation already unenrolled them and routed to FTHB LM2 - Roadmap)
---

# B6 — Day 21

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

## Polish notes
<!-- Capture edits, alternates, decisions here. -->

- [ ] Confirm Pipedrive Workflow Automation correctly handles the **cross-magnet unenroll** when `fthb_received_lm2 = true` flips mid-sequence (see `lm2-process-map/_index.md` for the email-storm rule)
- [ ] Optional: A/B the "back off" framing vs. a more confident sign-off

## Merge fields used
- `{{first_name}}`
- `{{book_bss_link}}`
- `{{agent_first_name}}`
