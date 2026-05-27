---
crm_name: "FTHB Quiz - Transactional - Day 00 - Bonus Delivery"
project: FTHB
feature: Quiz
magnet: LM1
tier: all
automation: "Email 0 - LM1 Transactional"
sequence_position: "Standalone one-step automation (fires for every LM1 submission, before tier automation)"
day: 0
delay_from_previous: "Immediate (within 60 seconds of form submit)"
subject: "Your {{tier_label}} bonus: 2 mistakes and a market snapshot"
preview: "The 2 mistakes buyers in your tier miss most, plus a 1-page Orlando market snapshot."
goal: "Confirm receipt, deliver the bonus (web page + PDF), set the tone."
---

# FTHB Quiz - Transactional - Day 00 - Bonus Delivery

**Subject:** Your {{tier_label}} bonus: 2 mistakes and a market snapshot

**Preview:** The 2 mistakes buyers in your tier miss most, plus a 1-page Orlando market snapshot.

**Send:** Immediately on form submit (within 60 seconds), via Make.com calling a one-step Pipedrive automation. Fires for every LM1 submission regardless of tier.

---

```
Hey *|FIRST_NAME|*,

Your Orlando First-Time Buyer Readiness Score came back as {{tier_label}}, {{display_score}}/100. Here's the bonus that goes with your tier.

It walks through your tier explanation, the 2 mistakes buyers in your tier most often make, and a 1-page Orlando market snapshot.

Two ways to read it. Same content. Pick whichever fits how you read.

👉 Read it on the web:
{{fthb_tier_bonus_page_link}}

👉 Download the PDF (save it, print it, mark it up):
{{fthb_tier_pdf_link}}

Two things to know:

1. I read every reply. If anything in the bonus raises a question (about your score, the market data, the next step, anything), just hit reply.
2. I'm not going to call you, text you, or pass your info to anyone. Not now, not ever. If we end up talking, it'll be because you booked something on your own.

-- Axel

Axel Rivera, REALTOR®
LPT Realty, LLC
(407) 227-3205

P.S. The 2-mistakes section gets a lot more useful the day before you actually talk to a lender, so bookmark the page or save the PDF and come back to it then.
```
