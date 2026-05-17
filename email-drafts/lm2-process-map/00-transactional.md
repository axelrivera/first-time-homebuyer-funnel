---
id: LM2-E0
funnel: fthb
magnet: lm2
campaign: FTHB LM2 - Roadmap
day: 0
trigger: Immediately on form submit (both `fthb_lm1_tier_b` and `fthb_lm2_standalone` sources)
goal: Deliver the PDF + web link; set expectations for follow-up
subject: "Your 9-Step Orlando Home Roadmap (link + PDF inside)"
preview_text: The full roadmap, plus where to start reading depending on where you are.
status: DRAFT
last_edited: 2026-05-17
source_doc: docs/07-process-map-emails.md
---

# Email 0 (LM2) — Transactional delivery

**Subject:** Your 9-Step Orlando Home Roadmap (link + PDF inside)
**Preview text:** The full roadmap, plus where to start reading depending on where you are.

```
Hey {{first_name}},

Here's the 9-Step First Home Roadmap as promised.

  → Read it on the web: {{fthb_roadmap_view_link}}
  → Download the PDF (for offline reading or printing):
    {{fthb_roadmap_pdf_link}}

If you took the Readiness Scorecard and landed in the 90-Day
Sprint tier, **start at Step 4** (lender pre-approval). That's
the bottleneck for almost everyone in that tier.

If you came in from one of my posts about the home-buying
process, start at the visual roadmap and find the step that
looks most like where you are.

Two notes before you dive in:

1. The roadmap is dense. That's intentional. Skim it once
   end-to-end first, then come back to the step you're in.

2. If a step raises a question, hit reply. I read every email
   and I usually respond within a day.

Talk soon,
{{agent_first_name}}

P.S. The roadmap link is a shareable URL. If you know someone
in Orlando who's a few steps behind you on this, send it. I'd
rather they get the map than figure it out the expensive way.
```

## Polish notes
<!-- Capture edits, alternates, decisions here. -->

- [ ] Confirm web link and PDF link both work for both source values
- [ ] Re-validate the "start at Step 4" guidance against the final LM2 step numbering

## Merge fields used
- `{{first_name}}`
- `{{fthb_roadmap_view_link}}`
- `{{fthb_roadmap_pdf_link}}`
- `{{agent_first_name}}`
