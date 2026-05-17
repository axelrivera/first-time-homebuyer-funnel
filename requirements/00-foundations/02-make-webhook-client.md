One module that knows the Make.com webhook URL and how to POST to it. Both LM1 and LM2 forms call into it; nothing else in the codebase knows the URL exists. This is the **only piece of new infrastructure** the funnel needs from the existing axelrivera.com site — `BaseLayout`, analytics, Tailwind, and the build pipeline are already there.

## Goal

A single module exports:

1. Typed payload definitions (`FthbLm1Payload`, `FthbLm2Payload`, and a union `FthbWebhookPayload`).
2. A `postToMake(payload: FthbWebhookPayload): void` function that fires a `fetch` POST and **does not await it**. The caller never waits for the response — the redirect to the next page happens immediately.

## Where the module lives

The conventional path is `src/lib/webhook.ts`, but if the existing site uses a different convention (`src/utils/`, `src/services/`, etc.), match that. **Do not create a new top-level directory just for this one file** — slot it into whatever the site already has.

## Funnel namespace

This funnel uses the slug `fthb` (first-time homebuyer). Every identifier that crosses into a globally-shared system — `magnet` and `source` payload values, Pipedrive custom field names, TypeScript types in this module — is prefixed `fthb_` so future funnels (sellers, investors, etc.) can coexist without collision.

JSON payload keys nested under `answers.*` and `scoring.*` are **not** prefixed (scoped by `magnet`); the Pipedrive field names storing those values **are** (`fthb_q1_credit_range` … `fthb_q10_lender`). Make.com handles that one-line mapping; this module doesn't need to know about it.

## Payload shapes (locked)

### LM1 payload

```json
{
  "magnet": "fthb_lm1",
  "submitted_at": "2026-05-14T18:32:11Z",
  "contact": {
    "name": "Maria Lopez",
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

- `magnet` is the literal string `"fthb_lm1"` (discriminator).
- `contact.phone` is nullable.
- `scoring.tier` is `"READY_NOW" | "NINETY_DAY" | "FOUNDATION"`.
- `scoring.applied_overrides` is an array of override ID strings (may be empty).

### LM2 payload

```json
{
  "magnet": "fthb_lm2",
  "submitted_at": "2026-05-14T18:45:00Z",
  "contact": {
    "name": "Carlos Lopez",
    "email": "carlos@example.com",
    "zip": "32750"
  },
  "source": "fthb_lm1_tier_b",
  "fthb_lm1_tier": "NINETY_DAY",
  "consent": true
}
```

- `magnet` is the literal string `"fthb_lm2"`.
- `source` is `"fthb_lm1_tier_b" | "fthb_lm2_standalone"`.
- `fthb_lm1_tier` is `"READY_NOW" | "NINETY_DAY" | "FOUNDATION" | null` (null when `source === "fthb_lm2_standalone"`).
- `consent` is always `true` at submission (the form gates submission on the checkbox).
- `utm_source` is nullable.

### Stable enum keys (LOCKED — Make.com routes on these; do not rename)

| Question              | Allowed values for `answers.qN_*`                                                          |
| --------------------- | ------------------------------------------------------------------------------------------ |
| `q1_credit_range`     | `'740_plus' \| '680_739' \| '620_679' \| '580_619' \| 'below_580' \| 'unknown'`            |
| `q2_credit_awareness` | `'full_report' \| 'score_seen' \| 'not_checked' \| 'dont_know'`                            |
| `q3_savings`          | `'40k_plus' \| '20k_40k' \| '10k_20k' \| '3k_10k' \| 'under_3k' \| 'none'`                 |
| `q4_savings_rate`     | `'1k_plus' \| '500_999' \| '200_499' \| 'under_200' \| 'none' \| 'drawing_down'`           |
| `q5_dti`              | `'none' \| 'under_10' \| '10_25' \| '25_40' \| 'over_40' \| 'unknown'`                     |
| `q6_revolving`        | `'never' \| 'sometimes' \| 'often' \| 'behind' \| 'no_cards'`                              |
| `q7_tenure`           | `'2_plus_years' \| '1_2_years' \| '6_12_months' \| 'under_6_months' \| 'between_jobs'`     |
| `q8_income_type`      | `'w2' \| 'w2_plus_1099' \| '1099_2plus_years' \| '1099_under_2' \| 'other'`                |
| `q9_timeline`         | `'30_days' \| '1_3_months' \| '3_6_months' \| '6_12_months' \| '1_2_years' \| 'exploring'` |
| `q10_lender`          | `'preapproved' \| 'spoken_no_letter' \| 'lender_in_mind' \| 'no_idea'`                     |

## Implementation notes

- **The webhook URL is a `const` at the top of the file.** The site has no env vars by design; bake the URL into the build. Add a comment block in the file explaining this so a future reader doesn't try to "fix" this by adding `.env`. The threat model is documented: Make.com webhooks are designed to be receivers, the URL is treated as semi-public, and if abuse becomes an issue the fix is to rotate the webhook URL and add a shared-token field to the payload — not introduce env vars or a backend.

  > **Note on existing site:** If axelrivera.com already uses `.env`, `import.meta.env`, or similar for _other_ configuration, that's fine — that's its own concern. The Make.com webhook URL specifically does **not** use env vars. One module, one hardcoded const.

- **Fire and forget.** `postToMake` calls:

  ```ts
  fetch(WEBHOOK_URL, {
    method: "POST",
    body: JSON.stringify(payload),
    keepalive: true,
    headers: { "Content-Type": "application/json" },
  });
  ```

  and intentionally **doesn't `await`** the result. Use `keepalive: true` so the request survives the page navigation that's about to happen.

- **Catch and swallow errors.** If `fetch` rejects (offline, CORS, anything), don't throw. The user has already earned the result page; a webhook failure must not block the redirect. Wrap the call in `try { ... } catch { /* swallow */ }`, or attach `.catch(() => {})` to the promise.

- **Type the payloads exactly** from the JSON examples above. Use the stable enum keys for answer values. These are locked — copy them verbatim from the table above.

- **`magnet` discriminator.** `FthbLm1Payload` has `magnet: 'fthb_lm1'`, `FthbLm2Payload` has `magnet: 'fthb_lm2'`. Pure discriminated union — TypeScript's narrowing makes the rest of the codebase pleasant to read.

- **Where the URL comes from.** The agent will provide the Make.com webhook URL out-of-band when this task is being executed. Stub `const WEBHOOK_URL = 'https://hook.us1.make.com/REPLACE_ME'` initially and surface a checklist item in the PR description so the agent replaces it before merge.

## Things NOT to do

- Don't put the URL in `.env`, `import.meta.env`, or any other indirection. It's a `const`.
- Don't `await postToMake(...)`. The whole point is fire-and-forget. If a future caller wants to `.then()` it, they can re-export the underlying `fetch` promise — but the public API stays sync-from-the-caller's-perspective.
- Don't add retry logic. Make.com's Google Sheet audit row is the recovery mechanism, not a retry loop.
- Don't add a "staging mode" flag that switches to a different URL. Use a separate scenario in Make.com if you need a non-production webhook; the URL is still hardcoded for that build.
- Don't put this module anywhere except the existing site's conventional shared-module directory (per audit notes).

## Definition of Done

- [ ] The module exists at the conventional shared-module path (per `EXISTING-SITE-NOTES.md` Section 2) and exports `postToMake` plus the payload types
- [ ] An LM1-shaped payload posted from a scratch page (or browser console) actually hits the Make.com webhook (verify in Make.com's execution log)
- [ ] Failing the network request (block in DevTools) doesn't throw; the calling page can still call `location.assign()` immediately after `postToMake()`
- [ ] TypeScript prevents mixing invalid letters into the enum keys (e.g., `q1_credit_range: 'A'` should be a compile error)

## Verification

```bash
# In the existing axelrivera.com repo:
npm run dev
# From the browser console on any page:
import('/src/lib/webhook.ts').then(m => m.postToMake({
  magnet: 'fthb_lm1',
  submitted_at: new Date().toISOString(),
  contact: { first_name: 'Test', email: 'test@example.com', zip: '32708', phone: null },
  answers: {
    q1_credit_range: '740_plus',
    q2_credit_awareness: 'full_report',
    q3_savings: '40k_plus',
    q4_savings_rate: '1k_plus',
    q5_dti: 'none',
    q6_revolving: 'never',
    q7_tenure: '2_plus_years',
    q8_income_type: 'w2',
    q9_timeline: '30_days',
    q10_lender: 'preapproved'
  },
  scoring: { raw_total: 89, display_score: 100, tier: 'READY_NOW', applied_overrides: [] }
}))
# Confirm in Make.com that a webhook execution fired.
```

If the existing site uses a path alias (`~/lib/webhook` instead of relative imports), use the alias. Match the existing convention.
