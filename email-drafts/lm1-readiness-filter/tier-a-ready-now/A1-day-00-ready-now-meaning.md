---
id: LM1-A1
funnel: fthb
magnet: lm1
tier: A (READY_NOW)
campaign: FTHB LM1 - Tier A
day: 0
trigger: ~10 minutes after transactional
goal: Frame "Ready Now" + introduce BSS
subject: One thing about your "Ready Now" score
status: DRAFT          # DRAFT | REVIEWING | POLISHED | SHIPPED
last_edited: 2026-05-17
source_doc: docs/04-readiness-filter-emails.md
---

# A1 — Day 0

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

## Polish notes
<!-- Capture edits, alternates, decisions here. -->

- [ ] Test: would a Ready Now prospect feel stupid to say no?
- [ ] Confirm corridor geography line still reads as authoritative, not generic

## Merge fields used
- `{{first_name}}`
- `{{book_bss_link}}`
- `{{agent_first_name}}`
