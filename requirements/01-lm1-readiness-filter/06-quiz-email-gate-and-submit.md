
Replace the post-Q10 placeholder with the email gate. On submit: run the scoring engine client-side, build the webhook payload, fire it at Make.com (fire-and-forget), redirect to the result page with plain query params.

## Prereqs

- [01-03 Scoring engine](./03-scoring-engine.md) (`computeScore`)
- [01-05 Wire up Q2–Q10](./05-quiz-all-10-questions.md)
- [00-02 Make.com webhook client](../00-foundations/02-make-webhook-client.md) (`postToMake` + `FthbLm1Payload` type)

## Why email is AFTER the questions (not before)

Putting the email last makes the questions feel like the value and the email feel like a small payment for the result they *already earned*. The result page renders inline anyway, so the email gate is for the **emailed copy + nurture**, not for unlocking the result.

## Goal

After Q10, the quiz shows an email-gate screen with first_name, email, ZIP, and optional phone. On submit, the page:

1. Validates the inputs.
2. Computes `tier` + `displayScore` with `computeScore(answers)`.
3. Builds the `FthbLm1Payload`.
4. Calls `postToMake(payload)` (fire-and-forget; doesn't await).
5. Immediately navigates to `/orlando-homebuying-readiness-quiz/result?n=...&t=A|B|C&s=...` via `location.assign()`.

## Files to modify

```
src/pages/orlando-homebuying-readiness-quiz/start.astro
src/scripts/quiz.ts
```

## Email gate screen (step 11)

Renders inside the same Astro page as the questions — same `<section hidden>` pattern. Heading example: "One last step — where should we send your result?" (keep it tight, agent's voice, never salesy).

### Fields

- `first_name` (required, text, min 1 char). Trim before submitting.
- `email` (required, type="email"). HTML5 validation + a manual regex to catch obvious junk (e.g., `/^[^\s@]+@[^\s@]+\.[^\s@]+$/`).
- `zip` (required, 5-digit US ZIP; pattern `^\d{5}$`). Helper text: "So we know you're in the Orlando metro."
- `phone` (optional, free-text; basic length check if provided). Helper text: "Optional. We won't call unless you book a session."

Do **not** validate the ZIP against an Orlando-metro list. Out-of-market ZIPs still get to see their result; the agent decides what to do with them in Pipedrive.

## LM1 payload schema (passed to `postToMake`)

```json
{
  "magnet": "fthb_lm1",
  "submitted_at": "2026-05-14T18:32:11Z",
  "contact": {
    "first_name": "Maria",
    "email": "maria@example.com",
    "zip": "32708",
    "phone": null
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
  }
}
```

- `submitted_at` is `new Date().toISOString()`.
- `contact.phone` is `null` if the user left it empty (not an empty string).
- `answers.qN_*` values are the stable enum keys (the radio `value` attributes from task 05).
- `scoring.*` comes from `computeScore(answers)` (note the camelCase → snake_case conversion: `rawTotal → raw_total`, `displayScore → display_score`, `appliedOverrides → applied_overrides`).

## Submit handler (in `quiz.ts`)

1. Prevent default form submit.
2. Validate. Show inline errors on bad fields; focus the first invalid one. Don't proceed.
3. Build the `Answers` object from the in-memory state.
4. Call `computeScore(answers)` → `{ rawTotal, displayScore, tier, appliedOverrides }`.
5. Build the `FthbLm1Payload` per the schema above.
6. Fire `track('fthb_readiness_email_gate_submit', { tier })` where `tier` is the URL letter (`'A' | 'B' | 'C'`).
7. Call `postToMake(payload)` — **don't `await`**. Wrap in try/catch and swallow; never block the redirect.
8. Write the LM2 Roadmap prefill bridge to `sessionStorage`:
   ```ts
   try {
     sessionStorage.setItem('fthb_prefill_first_name', firstName);
     sessionStorage.setItem('fthb_prefill_email', email);
   } catch { /* swallow — Safari private mode etc.; never block the redirect */ }
   ```
   These two keys are the contract with [02-03 Opt-in form](../02-lm2-process-map/03-opt-in-form.md). Write only `first_name` and `email` — never the quiz answers, the tier, or the score.
9. Map tier to URL letter: `READY_NOW → 'A'`, `NINETY_DAY → 'B'`, `FOUNDATION → 'C'`.
10. Build URL: `/orlando-homebuying-readiness-quiz/result?n=${encodeURIComponent(firstName)}&t=${letter}&s=${displayScore}`.
11. `window.location.assign(resultUrl)`.

## Analytics events this task wires

- `fthb_readiness_email_gate_shown` — fires when the email gate becomes the active step (in the state machine's `goTo` when target step is 11). No props.
- `fthb_readiness_email_gate_submit` — fires when the form submits after passing validation. Props: `{ tier: 'A' | 'B' | 'C' }`.

(Both events match what task 01-10 will verify.)

## Implementation notes

- **No double-submit.** Disable the submit button on click; if the redirect doesn't happen within 1 second (it should — `location.assign` is synchronous), re-enable so the user can retry.
- **Don't await the webhook.** The user has earned the result; a Make.com hiccup must not block them. The redirect happens immediately after kicking off the fetch.
- **No HMAC, no signed token, no expiry on the result URL.** The data is the user's own self-reported answers; Pipedrive holds the authoritative record. Tampering with the URL only changes what *they* see on *their* screen.

## Things NOT to do

- Don't add a "save and finish later" checkbox — there's nowhere to save it (no storage).
- Don't add a consent checkbox unless legal requires it. The act of submitting an opt-in form is implicit consent under most US frameworks; the Pipedrive unsubscribe link covers CAN-SPAM. If the agent later wants a checkbox, add it then — don't pre-add friction.
- Don't try to validate the ZIP against an Orlando-metro list.
- Don't store the answers in `localStorage` or `sessionStorage` "just in case the webhook fails." Make.com's Google Sheet fallback is the recovery; the client doesn't need state. The only `sessionStorage` write here is the LM2 Roadmap prefill bridge — `first_name` and `email` only, never the quiz answers.

## Definition of Done

- [ ] After Q10, the email gate renders as step 11
- [ ] Submitting an empty form shows inline errors and focuses the first invalid field
- [ ] Submitting a valid form fires the webhook (visible in Make.com's execution log)
- [ ] Submitting a valid form redirects to `/orlando-homebuying-readiness-quiz/result?n=...&t=A|B|C&s=...`
- [ ] If you block the webhook URL in DevTools Network, the redirect still happens immediately (the user is never stuck)
- [ ] `fthb_readiness_email_gate_shown` fires when step 11 becomes active
- [ ] `fthb_readiness_email_gate_submit` fires with `{ tier }` when the form submits successfully

## Verification

End-to-end with a Tier A-ish answer set:
```
Q1 = 740_plus, Q2 = full_report, Q3 = 40k_plus, Q4 = 1k_plus,
Q5 = none,    Q6 = never,        Q7 = 2_plus_years, Q8 = w2,
Q9 = 30_days, Q10 = preapproved
```
Submit with `first_name = "Test"`, `email = "test@example.com"`, `zip = "32708"`. Should land on `/orlando-homebuying-readiness-quiz/result?n=Test&t=A&s=100`. Webhook fires in parallel.
