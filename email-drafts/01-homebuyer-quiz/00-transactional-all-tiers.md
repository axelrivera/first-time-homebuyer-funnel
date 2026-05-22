---
crm_name: "FTHB Quiz - Transactional - Day 00 - Score Delivery"
project: FTHB
feature: Quiz
magnet: LM1
tier: all
automation: "Email 0 - LM1 Transactional"
sequence_position: "Standalone one-step automation (fires for every LM1 submission, before tier automation)"
day: 0
delay_from_previous: "Immediate (within 60 seconds of form submit)"
subject: "Your Orlando readiness score: {{tier_label}} ({{display_score}}/100)"
preview: "Your tier guide, the 2 mistakes to avoid, and your next step. All inside."
goal: "Confirm receipt, deliver the result link, set the tone."
---

# FTHB Quiz - Transactional - Day 00 - Score Delivery

**Subject:** Your Orlando readiness score: {{tier_label}} ({{display_score}}/100)

**Preview:** Your tier guide, the 2 mistakes to avoid, and your next step. All inside.

**Send:** Immediately on form submit (within 60 seconds), via Make.com calling a one-step Pipedrive automation. Fires for every LM1 submission regardless of tier.

---

```
Hey *|FIRST_NAME|*,

You came back as {{tier_label}}, {{display_score}}/100.

Here's your tier guide as a PDF. It walks through your tier explanation, the 2 mistakes buyers in your tier most often make, and a 1-page Orlando market snapshot:

{{fthb_tier_pdf_link}}

Your next step is on the last page.

Two things to know:

1. I read every reply. If anything in the guide raises a question (about your score, the market data, the next step, anything), just hit reply.
2. I'm not going to call you, text you, or pass your info to anyone. Not now, not ever. If we end up talking, it'll be because you booked something on your own.

Talk soon,

-- Axel

P.S. Save the PDF or print it. The 2-mistakes section gets a lot more useful the day before you actually talk to a lender, so come back to it then.
```
