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
[Quiz page: Q1 → Q10 + email gate, all in one route, in-memory state]
      │  (on final submit: POST to Make.com webhook in the background,
      │   compute tier/score client-side, build result URL)
      ▼
[Inline result page: tier + score + 2 mistakes + market snapshot + CTA]
      │  (driven by plain query params: ?n=Maria&t=B&s=68)
      ▼
[Make.com → Pipedrive Person → Pipedrive Workflow Automation
 → Pipedrive Campaigns transactional + nurture]
```

### Why email is captured AFTER the questions

Hormozi's effort/sacrifice principle: gates are friction. Putting the email last makes the questions feel like the value and the email feel like a small payment for the result they *already earned*. The result page renders inline anyway, so the email gate is for the **emailed copy + nurture**, not for unlocking the result.

### What happens if the user bounces before the email gate

Nothing is persisted. There is **no** cookie, `LocalStorage`, or `SessionStorage` resume — the site is cookie-free and storage-free by design. If the user closes the tab or reloads, they start the quiz over. The agent does not see incomplete submissions, ever.

If this turns out to hurt completion meaningfully in Phase 4, the iteration is to **shorten the quiz**, not to add storage. Storage adds compliance surface area we don't want.

---

## Routes & screens

All routes are under the agent's primary domain. Static Astro pages with vanilla JS for the interactive bits.

| Route | Screen | Notes |
|---|---|---|
| `/orlando-homebuying-readiness-quiz` | Landing page | Promo + start button. The hero CTA is the only thing on the page. Fully static, no JS required to render. |
| `/orlando-homebuying-readiness-quiz/start` | Quiz page | **One** Astro page that contains all 10 questions + the email gate. State is held in vanilla JS in memory; question switching is DOM show/hide, not page navigation. Progress bar, back button per question, no "save & exit." |
| `/orlando-homebuying-readiness-quiz/result` | Inline result page | Tier, score, market snapshot, 2 mistakes, CTA. Renders from plain query params (`?n=…&t=…&s=…`). See "Result rendering" below. Fully static; works with JS disabled (the params drive a server-rendered `if/else` in the `.astro` file). |
| `/orlando-homebuying-readiness-quiz/result?preview=A` (or `B`, `C`) | Preview mode | Same result page; the `preview` query param renders the named tier with stub data so the agent can review copy. No auth gate (the URL is publicly reachable but unlisted). Acceptable: nothing sensitive is gated. |

The previous `/start`-then-`/contact` two-route flow is intentionally collapsed into one page. Without storage, separate routes would lose state between Q10 and the email gate. One page, one in-memory state machine.

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
| `marketSnapshot` data | See `data/market-snapshot.json` | Already external; updates monthly |
| `marketSnapshot.anchorPrice` | $475,000 | The dollar figure used in the "What $X actually buys" framing on the result page. Should track the realistic FTHB budget in Orlando; calibrate against real submission income data in Phase 4. Lives in the same snapshot config as the per-area data. |
| Tier-result copy strings | See `03-readiness-filter-content.md` | Should live in i18n-ready locale files, not inline JSX/Astro |

**Rule of thumb when building:** if a value has a "this is our current guess" comment next to it, it belongs in config. If it's a structural constant (e.g., the number of questions), inline is fine.

**Note on the Make.com webhook URL.** The URL itself is baked into the build (the site has no env vars). Treat it as semi-public. It is OK for it to be visible in the page source; Make.com webhooks are designed to be receivers. If abuse becomes an issue, rotate the webhook URL and add a shared-token field to the payload that Make.com verifies before processing; that token also gets rebaked at build time. Do **not** introduce env vars, secrets management, or a backend to "protect" this URL — those are the wrong solution for the threat model.

This also applies to LM2 (see `05-process-map-spec.md`): default language and the "pause LM1 Tier B sequence on LM2 opt-in" behavior (implemented as a Pipedrive Workflow Automation) should both be configurable at the automation layer, not hardcoded in the static site.

---

## Form payload sent to Make.com

The Astro page POSTs a single JSON payload to the Make.com webhook (via `fetch`) once the user completes the email gate. The scoring engine runs in the browser before the POST, so `scoring` is computed client-side and included in the payload — Make.com trusts it. The `magnet: "lm1"` discriminator is required so the one shared webhook can fan out to the LM1 or LM2 scenario branch.

```json
{
  "magnet": "lm1",
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

Each `answers.qN_*` field uses a stable enum key (not the human-readable label) so the Make.com → Pipedrive mapping is reliable when copy gets edited later. The same enum keys are written to Pipedrive custom fields so Pipedrive Workflow Automations can route off them without re-parsing labels.

### What Make.com does with this payload

1. Write one row to the Google Sheet audit log (raw payload + timestamp). This is the fallback if Pipedrive errors.
2. Look up the contact in Pipedrive by email.
   - If not found: create a new Person with the contact + answer fields + `lm1_tier`, `lm1_display_score`, `received_lm1 = true`, `preferred_language`.
   - If found: update the existing Person with the new tier/score (retake handling) and set `lm1_retaken_at`.
3. Return `200` to the browser as fast as possible. The browser will have already redirected to the result page; the webhook response is fire-and-forget from the client's perspective.

Email sending and sequence enrollment are **not** Make.com's job. The moment Make.com sets the Pipedrive fields, a Pipedrive Workflow Automation matches on `received_lm1 = true` + `lm1_tier = NINETY_DAY` (etc.) and starts the corresponding Pipedrive Campaign sequence.

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

When the user submits the email gate, the quiz JS does two things in parallel:

1. Fires the Make.com `fetch` with the full payload (above). Does **not** await it.
2. Immediately redirects to `/orlando-homebuying-readiness-quiz/result?n=Maria&t=B&s=68`.

The result page reads three query params:

| Param | Meaning | Allowed values |
|---|---|---|
| `n` | First name (URL-encoded) | Any string; trimmed and HTML-escaped on render. Empty → render as "you" in copy. |
| `t` | Tier letter | `A`, `B`, `C`. Anything else → fall back to a generic "your result has been emailed to you" view. |
| `s` | Display score (0–100) | Integer 0–100. Anything else → tier-without-number view. |

**No HMAC, no signed token, no expiry.** The site has no environment variables, so there is no secret to sign with, and there is nothing on the page worth protecting:

- The data is the user's own self-reported answers.
- Pipedrive has the authoritative record (from the Make.com webhook).
- A user can tamper with their URL to display a different tier; nothing they can do with that affects the agent's CRM, the email sequence they're enrolled in, or which CTA gets the most warm lead. (If a tampered Tier-A page link were shared and someone booked the BSS, the agent qualifies in conversation; no harm done.)

The result page **never** trusts the URL for anything beyond rendering. No conditional that affects state (e.g., a "this person is Tier A, send them this thing") may key off the query params. State lives in Pipedrive.

The previous expiry-window scheme is gone — there is no concept of "result token expired" anymore. The result page works forever from a valid `?n=…&t=…&s=…` URL. The user receives the same URL in their transactional email from Pipedrive Campaigns; both reach the same static page.

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

Minimum viable event list. Emit these **directly from the browser** to the analytics provider's client script (Plausible, PostHog, or whatever Phase 0 lands on). Do **not** relay through Make.com — events should keep working even if the webhook is down.

| Event | When |
|---|---|
| `readiness_landing_view` | Landing page load |
| `readiness_quiz_start` | User clicks "Start" |
| `readiness_question_answered` | Each Q advances (props: q_id, answer_key) |
| `readiness_email_gate_shown` | Email gate becomes visible in the quiz page |
| `readiness_email_gate_submit` | Email gate `fetch` POST initiated (don't wait for the response) |
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
- [ ] All 10 questions render correctly on iOS Safari and Android Chrome (primary surface; QA gate). Question switching is in-page, no full page navigations.
- [ ] Desktop smoke check: no broken layouts and all interactions functional on a current-version desktop browser (not a polish gate)
- [ ] Scoring engine produces correct tier + score for the 6 canonical test cases (see `08-implementation-roadmap.md`)
- [ ] All 4 tier overrides fire correctly
- [ ] Form `fetch` POSTs to the Make.com webhook with the full payload above. Network failure on the POST does **not** block the redirect to the result page (the user has already earned the result).
- [ ] Make.com creates/updates a Pipedrive Person with `lm1_tier`, `lm1_display_score`, `received_lm1 = true`, and the answer fields
- [ ] Pipedrive Workflow Automation fires the tier-specific Pipedrive Campaign transactional email within 60 seconds of submit, for all 3 tiers
- [ ] Google Sheet audit row is written for every submission
- [ ] Result page renders all 3 tiers correctly from `?n=&t=A&s=…` / `?t=B` / `?t=C`
- [ ] Malformed query params fall back gracefully to the generic "your result has been emailed to you" view
- [ ] `?preview=A|B|C` renders the right tier with stub data
- [ ] Analytics events fire on every step from the client, not via Make.com
- [ ] Agent has taken the quiz themselves and confirmed copy/voice feels like theirs
