# 02 - LM1 Readiness Filter: Product Spec

This is the build spec for the scorecard funnel. Everything a developer needs to ship LM1 lives here: routes, screens, copy, the exact 10 questions, the scoring math, the tier thresholds, the form payload, and the result-page logic.

---

## Public name (final)

**The Orlando First-Time Buyer Readiness Score**
*Subhead:* Know in 7 Minutes If You're 30, 90, or 180 Days Away From Your First Home.

This wording is locked. Don't paraphrase it on landing pages, ads, or DMs. Copy/paste it.

---

## High-level user flow

```
[Landing page]
      │
      ▼
[Q1 → Q10  (one question per screen, with progress bar)]
      │
      ▼
[Email-capture gate: name + email + ZIP (Orlando area filter)]
      │
      ▼
[Inline result page: tier + score + 2 mistakes + market snapshot + CTA]
      │
      ▼
[Email follow-up sequence kicks off based on tier]
```

### Why email is captured AFTER the questions

Hormozi's effort/sacrifice principle: gates are friction. Putting the email last makes the questions feel like the value and the email feel like a small payment for the result they *already earned*. The result page renders inline anyway, so the email gate is for the **emailed copy + nurture**, not for unlocking the result.

If a prospect bounces at the email gate, the form session is saved client-side and resumed on return. The agent does not see incomplete submissions.

---

## Routes & screens

All routes are under the agent's primary domain. Astro pages.

| Route | Screen | Notes |
|---|---|---|
| `/orlando-homebuying-readiness-quiz` | Landing page | Promo + start button. The hero CTA is the only thing on the page. |
| `/orlando-homebuying-readiness-quiz/start` | Question screens (Q1–Q10) | Single-page Astro island. Progress bar. Back button on every question. No "save & exit". It's 7 minutes. |
| `/orlando-homebuying-readiness-quiz/contact` | Email-capture gate | Name (first), email, ZIP code. Optional: phone (not required). |
| `/orlando-homebuying-readiness-quiz/result` | Inline result page | Tier, score, market snapshot, 2 mistakes, CTA. Render driven by query params from the form submit response. See "Result rendering" below. |
| `/orlando-homebuying-readiness-quiz/result/preview` | Static preview (admin) | Lets the agent see all three tier pages without taking the quiz. Behind a basic-auth gate or unguessable URL. |

---

## Landing page (`/orlando-homebuying-readiness-quiz`)

Layout: above-the-fold = single hero, single CTA. Below-the-fold = social proof / FAQ.

### Hero copy

> **The Orlando First-Time Buyer Readiness Score**
> Know in 7 Minutes If You're 30, 90, or 180 Days Away From Your First Home.
>
> Answer 10 plain-English questions and get a personalized readiness score, with an exact timeline and the 2–3 specific things standing between you and your first Orlando home. No credit pull. No phone call. No "we'll be in touch."
>
> [ Start the 7-minute scorecard ]
>
> *Free. Built by a licensed Orlando agent. Bilingual support available.*

### Sub-hero. "What you'll get" (3-bullet, not a wall)

- A readiness tier. Ready Now, 90-Day Sprint, or Foundation Phase
- The 2 specific mistakes buyers in your tier make most often (so you don't)
- A 1-page Orlando market snapshot. What $475K actually buys along the I-4 corridor right now, with examples from Sanford and Lake Mary on the Seminole side and Winter Park and College Park on the Orange side

### FAQ (below the fold)

- *Will you pull my credit?* No. You self-report a range. We can't see anything.
- *Is this an ad to get me on the phone?* No. The result is yours. If you want to talk, there's a free strategy session at the end. Only if you want it.
- *Why 10 questions?* That's the minimum to give you a real answer. Anything less is a brochure.
- *Hablas español?* Sí, el resultado completo está disponible en español. *(Phase 3; link disabled until translation ships.)*

---

## The 10 Scorecard Questions

Each question is a single screen. Plain English. Single-select unless noted. Every option has a point value used by the scoring engine (see "Scoring math" below).

### Q1. Credit range

> **What's your best guess at your current credit score?**
> If you're not sure, pick the range that feels right. We're not pulling anything.

| Option | Points |
|---|---|
| 740 or higher | 10 |
| 680 – 739 | 8 |
| 620 – 679 | 5 |
| 580 – 619 | 2 |
| Below 580 | 0 |
| I have no idea | 1 |

**Disqualifier flag:** If "Below 580" OR "I have no idea" → set `credit_unknown_or_low = true`. This blocks Tier A regardless of total score (see "Tier overrides").

### Q2. Credit awareness

> **Have you looked at your credit report in the last 12 months?**

| Option | Points |
|---|---|
| Yes, I've looked at the full report and I know what's on it | 5 |
| I've seen a score (Credit Karma, my bank app, etc.) but not the full report | 3 |
| No, I haven't checked | 1 |
| I don't really know what a credit report is | 0 |

### Q3. Savings available

> **Roughly how much do you have saved that you could put toward buying a home?**
> This includes down payment, closing costs, and your reserve. Not your 401k.

| Option | Points |
|---|---|
| $40,000+ | 10 |
| $20,000 – $39,999 | 8 |
| $10,000 – $19,999 | 5 |
| $3,000 – $9,999 | 3 |
| Less than $3,000 | 1 |
| Nothing saved yet | 0 |

### Q4. Savings rate

> **How much are you adding to that savings each month right now?**

| Option | Points |
|---|---|
| $1,000 or more per month | 8 |
| $500 – $999 per month | 6 |
| $200 – $499 per month | 4 |
| Less than $200 per month | 2 |
| Nothing right now | 0 |
| I'm actually drawing down savings | 0 |

### Q5. Monthly debt load

> **Other than rent, how much of your monthly income goes to debt payments? (car, student loans, credit card minimums)**

| Option | Points |
|---|---|
| Nothing. I have no monthly debt payments | 10 |
| Less than 10% of my take-home pay | 8 |
| 10% – 25% of my take-home pay | 5 |
| 25% – 40% of my take-home pay | 2 |
| More than 40% | 0 |
| I don't know. I'd have to add it up | 1 |

### Q6. Revolving credit behavior

> **Do you carry a credit card balance from month to month?**

| Option | Points |
|---|---|
| Never. I pay the full balance every month | 8 |
| Sometimes. Once or twice a year | 5 |
| Often. Most months | 2 |
| I'm behind on at least one card right now | 0 |
| I don't use credit cards | 6 |

### Q7. Employment tenure

> **How long have you been at your current job (or current income source)?**

| Option | Points |
|---|---|
| 2+ years | 10 |
| 1 – 2 years | 7 |
| 6 – 12 months | 4 |
| Less than 6 months | 1 |
| Between jobs right now | 0 |

### Q8. Income type

> **How are you paid?**

| Option | Points |
|---|---|
| W-2 employee, salary or hourly | 8 |
| Mostly W-2, with some 1099/side income | 6 |
| 1099 or self-employed, 2+ years of tax returns | 5 |
| 1099 or self-employed, less than 2 years | 1 |
| Other (commission-only, gig only, between jobs) | 1 |

### Q9. Desired timeline

> **When do you ideally want to be in your first Orlando home?**

| Option | Points |
|---|---|
| Within the next 30 days | 10 |
| 1 – 3 months from now | 9 |
| 3 – 6 months from now | 7 |
| 6 – 12 months from now | 4 |
| 1 – 2 years from now | 2 |
| Just exploring, no timeline | 0 |

### Q10. Pre-approval status

> **Have you talked to a lender yet?**

| Option | Points |
|---|---|
| Yes. I have a pre-approval letter in hand | 10 |
| Yes. I've spoken with one but no letter yet | 7 |
| No, but I have a lender in mind | 3 |
| No, and I wouldn't know where to start | 0 |

---

## Scoring math

```
total_score = sum(points for each of Q1 through Q10)
max_possible = 89   (Q1=10 + Q2=5 + Q3=10 + Q4=8 + Q5=10 + Q6=8 + Q7=10 + Q8=8 + Q9=10 + Q10=10)
```

For display, normalize to a 100-point scale:

```
display_score = round((total_score / 89) * 100)
```

Note: the per-question max values were chosen based on the relative weight each question should carry in the readiness model, not to force the raw total to add to a round number. The display normalization handles the rest. Tier thresholds (below) are defined on the display score, so they are unaffected by the raw max.

### Tier thresholds (on `display_score`)

| Tier | Internal name | Range |
|---|---|---|
| A: Ready Now | `READY_NOW` | 75 – 100 |
| B: 90-Day Sprint | `NINETY_DAY` | 45 – 74 |
| C: Foundation Phase | `FOUNDATION` | 0 – 44 |

### Tier overrides (hard rules, applied AFTER the threshold lookup)

These override the score-based tier. They exist because a high total can mask a single deal-breaker.

1. **`credit_unknown_or_low == true`** → demote Tier A to Tier B. Reason: no responsible lender will pre-approve at sub-580, and "I have no idea" is functionally the same as "we can't underwrite this yet."
2. **Q9 = "Just exploring, no timeline"** → cap at Tier C, even with a high score. Reason: a buyer with no timeline is not a buyer yet. Putting them in Tier B and pitching LM2 will feel premature and burn trust.
3. **Q5 = "More than 40%"** → cap at Tier B. Reason: DTI over 40% will fail most front-end ratios; they need a debt-payoff plan, not a home-shopping plan.
4. **Q7 = "Between jobs right now"** → cap at Tier C. Reason: no employment = no underwriting, period.

Apply overrides in order. Once a tier has been demoted, do not re-promote it.

---

## Configuration: anything tweakable must live in config

**Principle:** any value in this spec that may need to be calibrated against real submission data (thresholds, point values, override rules, expiry windows, copy strings) must live in a single configuration source, not hardcoded inline in components or scoring functions. The goal is that adjusting a tweakable parameter is a one-line config change, not a code refactor.

Concretely, when building the scoring engine and result page, centralize the following in one config module (suggested: `src/config/readiness.ts` or a `readiness.config.json` imported by the scorer):

| Parameter | Default | Why it's tweakable |
|---|---|---|
| `tierThresholds.READY_NOW.min` | 75 | Calibrate to actual submission distribution in Phase 4 |
| `tierThresholds.NINETY_DAY.min` | 45 | Same. Likely to shift after first ~50 real submissions |
| `pointValues.q1` through `pointValues.q10` | Per the question tables above | If a question's options need to be reweighted later |
| `overrides.creditUnknownOrLow.action` | demote A → B | May want to widen to "demote A → C" if early data shows false positives |
| `overrides.exploringTimeline.action` | cap at C | Could relax to "cap at B" if exploring buyers convert better than expected |
| `overrides.highDTI.action` | cap at B | Same logic |
| `overrides.betweenJobs.action` | cap at C | Same logic |
| `resultToken.expiryDays` | 30 | Operational tweak; may go up or down based on email re-engagement patterns |
| `marketSnapshot` data | See `data/market-snapshot.json` | Already external; updates monthly |
| `marketSnapshot.anchorPrice` | $475,000 | The dollar figure used in the "What $X actually buys" framing on the result page. Should track the realistic FTHB budget in Orlando; calibrate against real submission income data in Phase 4. Lives in the same snapshot config as the per-area data. |
| Tier-result copy strings | See `03-readiness-filter-content.md` | Should live in i18n-ready locale files, not inline JSX/Astro |

**Rule of thumb when building:** if a value has a "this is our current guess" comment next to it, it belongs in config. If it's a structural constant (e.g., the number of questions), inline is fine.

This also applies to LM2 (see `05-process-map-spec.md`): token expiry, default language, and the "pause LM1 Tier B sequence on LM2 opt-in" behavior should all be config-driven, not hardcoded.

---

## Form payload sent to Make.com

The Astro form posts a single JSON payload to the Make.com webhook on the final email-capture step. Make.com handles email delivery, CRM logging, and any PDF generation.

```json
{
  "submitted_at": "2026-05-14T18:32:11Z",
  "contact": {
    "first_name": "Maria",
    "email": "maria@example.com",
    "zip": "32708",
    "phone": null,
    "preferred_language": "en"
  },
  "answers": {
    "q1_credit_range": "680_739",
    "q2_credit_awareness": "score_seen",
    "q3_savings": "10k_20k",
    "q4_savings_rate": "200_499",
    "q5_dti": "10_25",
    "q6_revolving": "never",
    "q7_tenure": "1_2_years",
    "q8_income_type": "w2",
    "q9_timeline": "3_6_months",
    "q10_lender": "spoken_no_letter"
  },
  "scoring": {
    "raw_total": 67,
    "display_score": 68,
    "tier": "NINETY_DAY",
    "applied_overrides": []
  },
  "source": {
    "utm_source": "instagram_bio",
    "utm_campaign": null,
    "referrer": "https://www.instagram.com/"
  }
}
```

Each `answers.qN_*` field uses a stable enum key (not the human-readable label) so Make.com routing is reliable when copy gets edited later.

### Stable enum keys (locked; don't rename)

| Question | Keys |
|---|---|
| q1_credit_range | `740_plus`, `680_739`, `620_679`, `580_619`, `below_580`, `unknown` |
| q2_credit_awareness | `full_report`, `score_seen`, `not_checked`, `dont_know` |
| q3_savings | `40k_plus`, `20k_40k`, `10k_20k`, `3k_10k`, `under_3k`, `none` |
| q4_savings_rate | `1k_plus`, `500_999`, `200_499`, `under_200`, `none`, `drawing_down` |
| q5_dti | `none`, `under_10`, `10_25`, `25_40`, `over_40`, `unknown` |
| q6_revolving | `never`, `sometimes`, `often`, `behind`, `no_cards` |
| q7_tenure | `2_plus_years`, `1_2_years`, `6_12_months`, `under_6_months`, `between_jobs` |
| q8_income_type | `w2`, `w2_plus_1099`, `1099_2plus_years`, `1099_under_2`, `other` |
| q9_timeline | `30_days`, `1_3_months`, `3_6_months`, `6_12_months`, `1_2_years`, `exploring` |
| q10_lender | `preapproved`, `spoken_no_letter`, `lender_in_mind`, `no_idea` |

---

## Result rendering

When the form submission succeeds, the user is redirected to `/orlando-homebuying-readiness-quiz/result?token=...`. The token is a short-lived signed value that encodes `tier` + `display_score` + `first_name`. The page is fully static, with no DB lookup. This keeps Make.com out of the render path.

Token payload (HMAC-signed with a shared secret in the Astro env):

```
{ "n": "Maria", "t": "NINETY_DAY", "s": 68, "exp": 1747244400 }
```

Token expires in 30 days. After expiry, the page falls back to a generic "your result has been emailed to you" view.

### What the result page contains, in order

1. **Header**: *"Maria, you're in the 90-Day Sprint tier."* Tier name + score badge.
2. **One-paragraph tier explanation** (from `03-readiness-filter-content.md`)
3. **Your timeline**. A single visual: where you are on a 0/30/90/180-day axis, with a marker
4. **The 2 mistakes you're most likely about to make** (tier-specific; content in `03-`)
5. **One-page Orlando market snapshot**. Embedded, not a separate file
6. **The "What's Next" CTA**. Tier-specific:
   - Tier A → **Book a Buyer Strategy Session** (calendar embed)
   - Tier B → **Get the 9-Step First Home Roadmap** (LM2 opt-in, prefilled with email)
   - Tier C → **Get the Foundation Phase nurture sequence** (already enrolled via Make.com; this is a confirmation, not a new opt-in)
7. **Email confirmation line**: *"We just emailed you a copy of this page. Check your spam if you don't see it in 2 minutes."*

---

## Analytics events to fire

Minimum viable event list. Provider TBD, but emit these from the Astro client and forward via Make.com:

| Event | When |
|---|---|
| `readiness_landing_view` | Landing page load |
| `readiness_quiz_start` | User clicks "Start" |
| `readiness_question_answered` | Each Q (props: q_id, answer_key) |
| `readiness_email_gate_view` | Email gate loads |
| `readiness_email_gate_submit` | Email gate POST succeeds |
| `readiness_result_view` | Result page loads (props: tier, score) |
| `readiness_cta_click` | CTA on result page clicked (props: tier, cta_target) |

---

## Target device, accessibility, and mobile baseline

**Mobile is the primary target.** The readiness filter is designed, built, and QA'd mobile-first. iOS Safari and Android Chrome on phone-sized viewports are the canonical experience. The funnel's traffic comes from Instagram, Facebook, and DMs, where ~all clicks resolve on a phone, so the design must be tuned to that **form factor and screen real estate** (single-column flow, thumb-reachable controls, short scrollable sections, no hover-dependent affordances). This is a responsive web experience, not a native app. "Mobile-first" here means the layout and interaction model are sized for a phone, not that we're emulating native UI patterns.

Desktop must work (no broken layouts, all interactions functional), but it is **not** a priority surface:

- Do not invest in desktop-specific layouts, multi-column variants, hover states, or wide-viewport polish beyond what falls out of a clean responsive scale-up.
- When a design or behavior trade-off pits mobile against desktop, mobile wins every time.
- Visual QA gate is mobile (iOS Safari + Android Chrome). Desktop gets a smoke check, nothing more.

Non-negotiables (apply on every device, mobile-first):

- One question per screen, mobile-first layout. Tap targets >= 44px.
- Progress bar at top (e.g., "3 of 10").
- Back button on every question. Never lose answers on back.
- Form fields use semantic HTML (`<fieldset>`, `<legend>`, `<label>`). Each option is a real `<input type="radio">`.
- Keyboard navigation works (Tab + Enter to submit each Q).
- Color is never the only signal for state (tier badges use color + label + icon).
- Screen-reader-only progress announcements as questions advance.

---

## Definition of Done: LM1 launch

LM1 ships when **all** of the following are true:

- [ ] Landing page is live at `/orlando-homebuying-readiness-quiz` and indexable
- [ ] All 10 question screens render correctly on iOS Safari and Android Chrome (primary surface; QA gate)
- [ ] Desktop smoke check: no broken layouts and all interactions functional on a current-version desktop browser (not a polish gate)
- [ ] Scoring engine produces correct tier + score for the 6 canonical test cases (see `08-implementation-roadmap.md`)
- [ ] All 4 tier overrides fire correctly
- [ ] Form posts to Make.com webhook with the full payload above
- [ ] Make.com delivers the tier-specific transactional email within 60 seconds of submit
- [ ] Result page renders tier A, B, and C correctly from a valid signed token
- [ ] Expired/invalid tokens fall back gracefully
- [ ] Analytics events fire on every step
- [ ] Agent has previewed all three tier result pages via `/orlando-homebuying-readiness-quiz/result/preview`
- [ ] Agent has taken the quiz themselves and confirmed copy/voice feels like theirs
