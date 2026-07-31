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
preview: "The 2 mistakes buyers in your tier miss most, plus an Orlando market snapshot."
goal: "Confirm receipt, deliver the bonus (web page), set the tone."
---

# FTHB Quiz - Transactional - Day 00 - Bonus Delivery

**Subject:** Your {{tier_label}} bonus: 2 mistakes and a market snapshot

**Preview:** The 2 mistakes buyers in your tier miss most, plus an Orlando market snapshot.

**Send:** Immediately on form submit (within 60 seconds), via Make.com calling a one-step Pipedrive automation. Fires for every LM1 submission regardless of tier.

---

```
Hi %contact_first_name%,

Your Orlando First-Time Buyer Readiness tier came back as {{tier_label}}. Here's the bonus that goes with it.

It walks through your tier explanation, the 2 mistakes buyers in your tier most often make, and an Orlando market snapshot.

👉 Read it on the web:
{{fthb_tier_bonus_page_link}}

I read every reply myself. If anything in there raises a question, about your tier, the market data, or what to do next, hit reply and ask me.

Best,

-%agent_first_name%

P.S. The 2-mistakes section gets a lot more useful the day before you actually talk to a lender, so bookmark the page and come back to it then.
```
