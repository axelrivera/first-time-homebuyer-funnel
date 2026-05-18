---
id: LM1-E0
funnel: fthb
magnet: lm1
tier: all
campaign: FTHB Readiness Quiz - Result Delivery (Email 0)
day: 0
trigger: Immediately on form submit (within 60 seconds)
goal: Confirm receipt, deliver the result link, set the tone
subject: "Your Orlando readiness score: {{tier_label}}"
preview_text: Score, timeline, and the 2 mistakes to avoid. All inside.
status: DRAFT          # DRAFT | REVIEWING | POLISHED | SHIPPED
last_edited: 2026-05-17
source_doc: docs/04-readiness-filter-emails.md
---

# Email 0 — Transactional delivery (all tiers)

**Subject:** Your Orlando readiness score: {{tier_label}}
**Preview text:** Score, timeline, and the 2 mistakes to avoid. All inside.

```
Hey {{first_name}},

You came back as {{tier_label}}, {{display_score}}/100.

Here's your tier guide as a PDF. It walks through your tier
explanation, the 2 mistakes buyers in your tier most often
make, and a 1-page Orlando market snapshot:

    {{fthb_tier_pdf_link}}

Your next step is on the last page.

Two things to know:

1. I read every reply. If anything in the guide raises a
   question (about your score, the market data, the next step,
   anything), just hit reply.

2. I'm not going to call you, text you, or pass your info to
   anyone. Not now, not ever. If we end up talking, it'll be
   because you booked something on your own.

Talk soon,
{{agent_first_name}}
{{agent_license_no}} | {{brokerage}}

P.S. Save the PDF or print it. The 2-mistakes section gets a
lot more useful the day before you actually talk to a lender,
so come back to it then.
```

## Polish notes
<!-- Use this section to capture your edits, alternates, and decisions. -->

- [ ] Confirm `{{tier_label}}` displays as "Ready Now" / "90-Day Sprint" / "Foundation Phase" (not enum keys)
- [ ] Confirm `{{fthb_tier_pdf_link}}` resolves to the correct per-tier PDF (Tier A / B / C) based on `fthb_lm1_tier`
- [ ] Decide PDF hosting + URL convention (e.g., `/pdfs/fthb-lm1-tier-a.pdf`) and lock filenames before sequence goes live
- [ ] Confirm each tier PDF ends with the next-step page referenced in the body
- [ ] Verify P.S. hook still lands after a polish pass
- [ ] Reconcile with [docs/04-readiness-filter-emails.md](../../docs/04-readiness-filter-emails.md) and [docs/02-readiness-filter-spec.md](../../docs/02-readiness-filter-spec.md): result-page-as-delivery is being replaced by a tier PDF here; specs need an update pass

## Merge fields used
- `{{first_name}}`
- `{{tier_label}}`
- `{{display_score}}`
- `{{fthb_tier_pdf_link}}` — resolved from `fthb_lm1_tier` to the matching per-tier PDF URL (Tier A / B / C)
- `{{agent_first_name}}`
- `{{agent_license_no}}`
- `{{brokerage}}`
