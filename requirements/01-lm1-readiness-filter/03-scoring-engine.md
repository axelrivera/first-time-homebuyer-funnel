
A pure TypeScript function that turns a 10-question answer set into a tier and a display score, reproducing the 6 canonical scoring examples below.

## Goal

`src/lib/scoring.ts` exports a pure function:

```ts
function computeScore(answers: Answers): ScoreResult
```

where `Answers` is the 10-question shape (keyed by `q1_credit_range` ... `q10_lender`, valued by the stable enum keys) and `ScoreResult` is:

```ts
type ScoreResult = {
  rawTotal: number;
  displayScore: number;
  tier: 'READY_NOW' | 'NINETY_DAY' | 'FOUNDATION';
  appliedOverrides: string[];  // override `id` values, in the order applied
};
```

The function reads config from `src/config/readiness.ts` (does not hardcode point values, thresholds, or override rules).

## Files to create

```
src/lib/scoring.ts
```

## Order of operations (do not shortcut)

1. **Sum raw points** for each of Q1–Q10 from `POINT_VALUES[qN][answer]`. That's `rawTotal`. If any answer key is missing or unknown, **throw** — a missing answer is a programmer bug, not something to silently default.
2. **Normalize** to display: `displayScore = Math.round((rawTotal / RAW_MAX) * 100)`. That's the number the user sees.
3. **Initial tier from thresholds:**
   ```
   displayScore >= TIER_THRESHOLDS.READY_NOW.min   → 'READY_NOW'
   displayScore >= TIER_THRESHOLDS.NINETY_DAY.min  → 'NINETY_DAY'
   otherwise                                        → 'FOUNDATION'
   ```
4. **Walk `OVERRIDES` in array order.** For each override whose predicate matches the answers:
   - Compute the action's resulting tier:
     - `demote_a_to_b`: if current tier is `READY_NOW`, becomes `NINETY_DAY`; else unchanged.
     - `cap_at_b`: tier becomes `min(currentRank, NINETY_DAY rank)` (i.e., `READY_NOW` becomes `NINETY_DAY`; `NINETY_DAY` and `FOUNDATION` unchanged).
     - `cap_at_c`: tier becomes `FOUNDATION`.
   - **Apply only if the resulting tier has lower rank than the current tier.** Never re-promote.
   - Push the override's `id` into `appliedOverrides` whenever its predicate matches, regardless of whether the action actually changed the tier (a downstream caller may want to know that an override fired even on an already-low tier — and the spec calls this out explicitly).
5. Return `{ rawTotal, displayScore, tier, appliedOverrides }`.

Tier ranks: `READY_NOW=2, NINETY_DAY=1, FOUNDATION=0`. Implement explicitly so it's auditable from the code.

## The 6 canonical scoring examples (engine must reproduce)

| Case | Answers | Expected tier | Expected behavior |
|---|---|---|---|
| 1 | All-high: `q1_credit_range='740_plus'`, `q2='full_report'`, `q3='40k_plus'`, `q4='1k_plus'`, `q5='none'`, `q6='never'`, `q7='2_plus_years'`, `q8='w2'`, `q9='30_days'`, `q10='preapproved'` | `READY_NOW` | `rawTotal === 89`, `displayScore === 100`, `appliedOverrides === []` |
| 2 | Mid-range: `q1='680_739'`, `q2='score_seen'`, `q3='10k_20k'`, `q4='200_499'`, `q5='10_25'`, `q6='never'`, `q7='1_2_years'`, `q8='w2'`, `q9='3_6_months'`, `q10='lender_in_mind'` | `NINETY_DAY` | `displayScore` in [55, 75] |
| 3 | Low everything: `q1='580_619'`, `q2='not_checked'`, `q3='under_3k'`, `q4='none'`, `q5='25_40'`, `q6='often'`, `q7='under_6_months'`, `q8='other'`, `q9='1_2_years'`, `q10='no_idea'` | `FOUNDATION` | `displayScore < 30` |
| 4 | High score except `q1_credit_range='unknown'` (use Case 1 answers, swap Q1) | `NINETY_DAY` | `appliedOverrides` includes `'creditUnknownOrLow'` |
| 5 | High score except `q9_timeline='exploring'` (use Case 1 answers, swap Q9) | `FOUNDATION` | `appliedOverrides` includes `'exploringTimeline'` |
| 6 | High score except `q7_tenure='between_jobs'` (use Case 1 answers, swap Q7) | `FOUNDATION` | `appliedOverrides` includes `'betweenJobs'` |

Walk through each by hand once the engine compiles. Cases 4–6 demonstrate that a single deal-breaker overrides the score-based tier.

## Implementation notes

- **Pure function.** No DOM, no `window`, no side effects. The same input always produces the same output.
- **Reads from config.** Import `POINT_VALUES`, `TIER_THRESHOLDS`, `OVERRIDES`, `RAW_MAX` from `src/config/readiness.ts`. No magic numbers.
- **Override predicates are data.** The config defines each predicate as something like `{ qid: 'q1_credit_range', values: ['below_580', 'unknown'] }`. The engine evaluates this generically: `answers[predicate.qid] && predicate.values.includes(answers[predicate.qid])`. No `if/else` ladders keyed off override `id`.
- **Track `appliedOverrides` in order applied.** Even if multiple overrides fire, both go in the list; the final tier is the lowest rank produced.

## Things NOT to do

- Don't import the UI, the result page, or anything from `src/pages/`. This function is engine-only.
- Don't re-promote a tier after a demote/cap. If two overrides fire, both go in `appliedOverrides`, but the final tier is the lowest rank produced.
- Don't write a default case that silently returns `'FOUNDATION'` if a question is missing. Throw — a missing answer is a programmer bug.
- Don't bake any point values, thresholds, or override predicates into the engine source. Everything reads from config.

## Definition of Done

- [ ] All 6 canonical scoring examples reproduce by hand (run them through `computeScore` from a scratch page or the browser console; confirm tier and overrides).
- [ ] Each of the 4 overrides fires when its predicate matches and stays silent otherwise — verified by hand against representative submissions.
- [ ] When two overrides fire on the same submission, the final tier is the lowest rank produced and both names appear in `appliedOverrides`.
- [ ] Missing or unknown answer keys throw at compute time.
- [ ] No imports from `astro:` or `src/pages/` — the engine is UI-independent.
