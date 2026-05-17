
One file that holds every tweakable parameter for the scoring engine: thresholds, point values, override rules. The scoring engine in the next task imports from here and hardcodes nothing.

## Goal

`src/config/readiness.ts` exports a typed config object with:

- Per-question point-value maps (Q1 through Q10, keyed by stable enum keys)
- Tier thresholds (`READY_NOW.min`, `NINETY_DAY.min`) on the display score
- Override rules (4 of them, in spec order)
- The `RAW_MAX` constant (89) and `DISPLAY_MAX` (100) — structural constants but worth naming

The scoring engine in [03-scoring-engine.md](./03-scoring-engine.md) will consume this. Nothing in the config file *does* anything — it's pure data.

## Files to create

```
src/config/readiness.ts
src/types/readiness.ts   (optional; types can live in the config file if small)
```

## The 10 questions with point values

Every option in every question table maps to its stable enum key with a point value. **The stable enum keys are LOCKED** — Make.com and Pipedrive route on them. Do not rename.

### Q1 — `q1_credit_range`

| Stable key | UI label | Points |
|---|---|---|
| `740_plus` | 740 or higher | 10 |
| `680_739` | 680 – 739 | 8 |
| `620_679` | 620 – 679 | 5 |
| `580_619` | 580 – 619 | 2 |
| `below_580` | Below 580 | 0 |
| `unknown` | I have no idea | 1 |

### Q2 — `q2_credit_awareness`

| Stable key | UI label | Points |
|---|---|---|
| `full_report` | Yes, I've looked at the full report and I know what's on it | 5 |
| `score_seen` | I've seen a score (Credit Karma, my bank app, etc.) but not the full report | 3 |
| `not_checked` | No, I haven't checked | 1 |
| `dont_know` | I don't really know what a credit report is | 0 |

### Q3 — `q3_savings`

| Stable key | UI label | Points |
|---|---|---|
| `40k_plus` | $40,000+ | 10 |
| `20k_40k` | $20,000 – $39,999 | 8 |
| `10k_20k` | $10,000 – $19,999 | 5 |
| `3k_10k` | $3,000 – $9,999 | 3 |
| `under_3k` | Less than $3,000 | 1 |
| `none` | Nothing saved yet | 0 |

### Q4 — `q4_savings_rate`

| Stable key | UI label | Points |
|---|---|---|
| `1k_plus` | $1,000 or more per month | 8 |
| `500_999` | $500 – $999 per month | 6 |
| `200_499` | $200 – $499 per month | 4 |
| `under_200` | Less than $200 per month | 2 |
| `none` | Nothing right now | 0 |
| `drawing_down` | I'm actually drawing down savings | 0 |

### Q5 — `q5_dti`

| Stable key | UI label | Points |
|---|---|---|
| `none` | Nothing. I have no monthly debt payments | 10 |
| `under_10` | Less than 10% of my take-home pay | 8 |
| `10_25` | 10% – 25% of my take-home pay | 5 |
| `25_40` | 25% – 40% of my take-home pay | 2 |
| `over_40` | More than 40% | 0 |
| `unknown` | I don't know. I'd have to add it up | 1 |

### Q6 — `q6_revolving`

| Stable key | UI label | Points |
|---|---|---|
| `never` | Never. I pay the full balance every month | 8 |
| `sometimes` | Sometimes. Once or twice a year | 5 |
| `often` | Often. Most months | 2 |
| `behind` | I'm behind on at least one card right now | 0 |
| `no_cards` | I don't use credit cards | 6 |

### Q7 — `q7_tenure`

| Stable key | UI label | Points |
|---|---|---|
| `2_plus_years` | 2+ years | 10 |
| `1_2_years` | 1 – 2 years | 7 |
| `6_12_months` | 6 – 12 months | 4 |
| `under_6_months` | Less than 6 months | 1 |
| `between_jobs` | Between jobs right now | 0 |

### Q8 — `q8_income_type`

| Stable key | UI label | Points |
|---|---|---|
| `w2` | W-2 employee, salary or hourly | 8 |
| `w2_plus_1099` | Mostly W-2, with some 1099/side income | 6 |
| `1099_2plus_years` | 1099 or self-employed, 2+ years of tax returns | 5 |
| `1099_under_2` | 1099 or self-employed, less than 2 years | 1 |
| `other` | Other (commission-only, gig only, between jobs) | 1 |

### Q9 — `q9_timeline`

| Stable key | UI label | Points |
|---|---|---|
| `30_days` | Within the next 30 days | 10 |
| `1_3_months` | 1 – 3 months from now | 9 |
| `3_6_months` | 3 – 6 months from now | 7 |
| `6_12_months` | 6 – 12 months from now | 4 |
| `1_2_years` | 1 – 2 years from now | 2 |
| `exploring` | Just exploring, no timeline | 0 |

### Q10 — `q10_lender`

| Stable key | UI label | Points |
|---|---|---|
| `preapproved` | Yes. I have a pre-approval letter in hand | 10 |
| `spoken_no_letter` | Yes. I've spoken with one but no letter yet | 7 |
| `lender_in_mind` | No, but I have a lender in mind | 3 |
| `no_idea` | No, and I wouldn't know where to start | 0 |

## Scoring math

```
RAW_MAX = 89   (Q1=10 + Q2=5 + Q3=10 + Q4=8 + Q5=10 + Q6=8 + Q7=10 + Q8=8 + Q9=10 + Q10=10)
DISPLAY_MAX = 100
display_score = round((raw_total / 89) * 100)
```

Tier thresholds (on the display score):

| Tier (internal name) | Range |
|---|---|
| `READY_NOW` (A: Ready Now) | 75 – 100 |
| `NINETY_DAY` (B: 90-Day Sprint) | 45 – 74 |
| `FOUNDATION` (C: Foundation Phase) | 0 – 44 |

## Tier overrides (LOCKED order; engine applies them in this sequence)

Override actions: `'demote_a_to_b'` | `'cap_at_b'` | `'cap_at_c'`. Tier rank: `READY_NOW=2, NINETY_DAY=1, FOUNDATION=0`. **Never re-promote** once an override has demoted/capped.

1. **`creditUnknownOrLow`** — fires when `q1_credit_range` is `'below_580'` or `'unknown'`. Action: `demote_a_to_b`. Reason: no responsible lender will pre-approve at sub-580, and "I have no idea" is functionally the same as "we can't underwrite this yet."
2. **`highDTI`** — fires when `q5_dti` is `'over_40'`. Action: `cap_at_b`. Reason: DTI over 40% will fail most front-end ratios; they need a debt-payoff plan, not a home-shopping plan.
3. **`exploringTimeline`** — fires when `q9_timeline` is `'exploring'`. Action: `cap_at_c`. Reason: a buyer with no timeline is not a buyer yet. Putting them in Tier B and pitching LM2 will feel premature and burn trust.
4. **`betweenJobs`** — fires when `q7_tenure` is `'between_jobs'`. Action: `cap_at_c`. Reason: no employment = no underwriting, period.

## What's tweakable (lives here) vs. structural (inline is fine)

| Parameter | Default | Why it's tweakable |
|---|---|---|
| `TIER_THRESHOLDS.READY_NOW.min` | 75 | Calibrate to actual submission distribution in Phase 4 |
| `TIER_THRESHOLDS.NINETY_DAY.min` | 45 | Same. Likely to shift after first ~50 real submissions |
| `POINT_VALUES.qN.*` | Per tables above | If a question's options need to be reweighted later |
| `OVERRIDES.creditUnknownOrLow.action` | `demote_a_to_b` | May want to widen to `cap_at_c` if early data shows false positives |
| `OVERRIDES.exploringTimeline.action` | `cap_at_c` | Could relax to `cap_at_b` if exploring buyers convert better than expected |
| `OVERRIDES.highDTI.action` | `cap_at_b` | Same logic |
| `OVERRIDES.betweenJobs.action` | `cap_at_c` | Same logic |

Structural constants like "there are 10 questions" can be inline.

## Implementation notes

- Use TypeScript `as const` so the enum-key strings narrow to literal types. Example:

  ```ts
  export const Q1_POINTS = {
    '740_plus': 10,
    '680_739': 8,
    '620_679': 5,
    '580_619': 2,
    'below_580': 0,
    'unknown': 1,
  } as const;
  ```

- Export a discriminated `Override` type. Each override has `id`, `predicate` (described by data, not a function — the engine interprets it: e.g., `{ qid: 'q1_credit_range', values: ['below_580', 'unknown'] }`), and `action` (`'demote_a_to_b' | 'cap_at_b' | 'cap_at_c'`).
- `OVERRIDES` is an **array** in the order above. Order is load-bearing.
- Add a header comment listing what's tweakable here and what isn't. Mirror the table above.

## Things NOT to do

- Don't put any logic in this file. The override rules are described as data; the engine interprets them. Mixing the two makes calibration in Phase 4 painful.
- Don't add token-expiry config — there are no signed tokens; result URLs use plain query params.
- Don't add `preferred_language` or language config — bilingual is out for the initial release.
- Don't rename any stable enum key. Make.com → Pipedrive mapping routes on these.

## Definition of Done

- [ ] `src/config/readiness.ts` exports `RAW_MAX = 89`, `DISPLAY_MAX = 100`, `TIER_THRESHOLDS`, `POINT_VALUES` (10 question maps), `OVERRIDES` (4-item array in the order above)
- [ ] Every stable enum key from the question tables appears as a config key (no typos, no rename)
- [ ] TypeScript compiles with strict mode
- [ ] `OVERRIDES` is an array (preserves order) — not an object map
- [ ] The file has a header comment listing what's tweakable here and what isn't

## Verification

```bash
npx tsc --noEmit  # strict mode passes
```

Eyeball-diff each point-value map against the tables above. One transposed digit here means the scoring engine produces wrong tiers.
