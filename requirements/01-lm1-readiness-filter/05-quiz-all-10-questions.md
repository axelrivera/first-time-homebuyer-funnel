
Fill in the remaining 9 questions in the quiz state machine. Each one is a `QuizQuestion` instance with copy from the canonical text below and options keyed by the stable enum keys from the config module.

## Goal

After this task, the quiz has all 10 questions visible (one at a time, in order) and the state machine advances from Q1 through Q10 correctly. The screen after Q10 is still a placeholder — the email gate lands in the next task.

## Files to modify

```
src/pages/orlando-homebuying-readiness-quiz/start.astro
```

Possibly minor edits to:
```
src/scripts/quiz.ts   (only if Q-specific handling is needed; ideally not)
```

## Canonical question copy (ship verbatim)

Every question is single-select. There's no free-text input until the email gate (next task). The stable enum key is the `value` attribute on each radio input. UI labels can be re-copy-edited later if needed, but the enum keys are locked.

### Q1 — `q1_credit_range` (already wired in 01-04)

> **What's your best guess at your current credit score?**
> If you're not sure, pick the range that feels right. We're not pulling anything.

| `value` | UI label |
|---|---|
| `740_plus` | 740 or higher |
| `680_739` | 680 – 739 |
| `620_679` | 620 – 679 |
| `580_619` | 580 – 619 |
| `below_580` | Below 580 |
| `unknown` | I have no idea |

### Q2 — `q2_credit_awareness`

> **Have you looked at your credit report in the last 12 months?**

| `value` | UI label |
|---|---|
| `full_report` | Yes, I've looked at the full report and I know what's on it |
| `score_seen` | I've seen a score (Credit Karma, my bank app, etc.) but not the full report |
| `not_checked` | No, I haven't checked |
| `dont_know` | I don't really know what a credit report is |

### Q3 — `q3_savings`

> **Roughly how much do you have saved that you could put toward buying a home?**
> This includes down payment, closing costs, and your reserve. Not your 401k.

| `value` | UI label |
|---|---|
| `40k_plus` | $40,000+ |
| `20k_40k` | $20,000 – $39,999 |
| `10k_20k` | $10,000 – $19,999 |
| `3k_10k` | $3,000 – $9,999 |
| `under_3k` | Less than $3,000 |
| `none` | Nothing saved yet |

### Q4 — `q4_savings_rate`

> **How much are you adding to that savings each month right now?**

| `value` | UI label |
|---|---|
| `1k_plus` | $1,000 or more per month |
| `500_999` | $500 – $999 per month |
| `200_499` | $200 – $499 per month |
| `under_200` | Less than $200 per month |
| `none` | Nothing right now |
| `drawing_down` | I'm actually drawing down savings |

### Q5 — `q5_dti`

> **Other than rent, how much of your monthly income goes to debt payments? (car, student loans, credit card minimums)**

| `value` | UI label |
|---|---|
| `none` | Nothing. I have no monthly debt payments |
| `under_10` | Less than 10% of my take-home pay |
| `10_25` | 10% – 25% of my take-home pay |
| `25_40` | 25% – 40% of my take-home pay |
| `over_40` | More than 40% |
| `unknown` | I don't know. I'd have to add it up |

(`over_40` is an override trigger — engine caps at Tier B.)

### Q6 — `q6_revolving`

> **Do you carry a credit card balance from month to month?**

| `value` | UI label |
|---|---|
| `never` | Never. I pay the full balance every month |
| `sometimes` | Sometimes. Once or twice a year |
| `often` | Often. Most months |
| `behind` | I'm behind on at least one card right now |
| `no_cards` | I don't use credit cards |

### Q7 — `q7_tenure`

> **How long have you been at your current job (or current income source)?**

| `value` | UI label |
|---|---|
| `2_plus_years` | 2+ years |
| `1_2_years` | 1 – 2 years |
| `6_12_months` | 6 – 12 months |
| `under_6_months` | Less than 6 months |
| `between_jobs` | Between jobs right now |

(`between_jobs` is an override trigger — engine caps at Tier C.)

### Q8 — `q8_income_type`

> **How are you paid?**

| `value` | UI label |
|---|---|
| `w2` | W-2 employee, salary or hourly |
| `w2_plus_1099` | Mostly W-2, with some 1099/side income |
| `1099_2plus_years` | 1099 or self-employed, 2+ years of tax returns |
| `1099_under_2` | 1099 or self-employed, less than 2 years |
| `other` | Other (commission-only, gig only, between jobs) |

### Q9 — `q9_timeline`

> **When do you ideally want to be in your first Orlando home?**

| `value` | UI label |
|---|---|
| `30_days` | Within the next 30 days |
| `1_3_months` | 1 – 3 months from now |
| `3_6_months` | 3 – 6 months from now |
| `6_12_months` | 6 – 12 months from now |
| `1_2_years` | 1 – 2 years from now |
| `exploring` | Just exploring, no timeline |

(`exploring` is an override trigger — engine caps at Tier C.)

### Q10 — `q10_lender`

> **Have you talked to a lender yet?**

| `value` | UI label |
|---|---|
| `preapproved` | Yes. I have a pre-approval letter in hand |
| `spoken_no_letter` | Yes. I've spoken with one but no letter yet |
| `lender_in_mind` | No, but I have a lender in mind |
| `no_idea` | No, and I wouldn't know where to start |

## Implementation notes

- **Copy verbatim.** Every question prompt, help text, and option label uses the exact wording above. The labels are user-facing; the stable enum keys are the `value` attribute on each radio (e.g., `<input type="radio" name="q1_credit_range" value="680_739">`). Don't invent new keys; reuse the ones from `src/config/readiness.ts`.
- **Q1 is already wired** from task 04. Add Q2 through Q10 as 9 more `<QuizQuestion ...>` invocations.
- **Question shape consistency.** Every question is single-select. There's no free-text input until the email gate (next task). Note the unusual point value on Q6 `no_cards` (6 pts) — the engine handles that, not this task.
- **Progress bar** must read "Question X of 10" accurately as the user advances. With 10 questions wired up, the email-gate screen in the next task becomes step 11 with a different label ("One last thing").
- Don't change the per-step DOM structure; keep using the `QuizQuestion.astro` component so the state machine's show/hide logic continues to work without changes.

## Question reference (cross-check option counts)

| Step | qid | Options count | Override trigger? |
|---|---|---|---|
| 1 | q1_credit_range | 6 | `below_580` / `unknown` → demote A→B (handled in engine) |
| 2 | q2_credit_awareness | 4 | — |
| 3 | q3_savings | 6 | — |
| 4 | q4_savings_rate | 6 | — |
| 5 | q5_dti | 6 | `over_40` → cap at B |
| 6 | q6_revolving | 5 | — |
| 7 | q7_tenure | 5 | `between_jobs` → cap at C |
| 8 | q8_income_type | 5 | — |
| 9 | q9_timeline | 6 | `exploring` → cap at C |
| 10 | q10_lender | 4 | — |

The UI doesn't need to know about override triggers — the engine runs after the email gate.

## Things NOT to do

- Don't paraphrase the prompts or the help text. The Q&A copy is the value proposition — generic wording erodes the diagnostic feel.
- Don't add a "Skip this question" button. Every question is required. If the user wants out, they close the tab.
- Don't pre-select a default option. Each question starts unanswered; the Next button stays disabled until something is selected.
- Don't bundle multiple questions on one screen even on desktop. The single-question pattern is part of the "feels diagnostic, not promotional" UX.

## Definition of Done

- [ ] All 10 questions visible in order, one at a time, with copy matching the text above verbatim
- [ ] Each option's `value` attribute uses the stable enum key from `src/config/readiness.ts`
- [ ] Next button is disabled until an option is selected
- [ ] Back works on every question (Q1 through Q10); answers persist when navigating back
- [ ] After Q10's Next, the screen shows a placeholder (e.g., "Email gate coming next") — task 06 replaces this
- [ ] Progress bar reads correctly at every step
- [ ] `fthb_readiness_question_answered` fires on every forward advance, with `q_id` and `answer_key` props

## Verification

Run the quiz top-to-bottom on a phone-sized viewport. Use back buttons to go back to earlier questions; their answers should still be selected. Watch the analytics events feed — should see 10 `fthb_readiness_question_answered` events in order.
