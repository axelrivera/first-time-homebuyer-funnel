
The opt-in form that turns a roadmap reader into a Pipedrive contact in the LM2 nurture sequence. Pre-fills from URL params (`?n=`, `?e=`, `?src=`) when present; otherwise falls back to `sessionStorage` (written by the LM1 submit handler) so a user arriving from a Tier B result page in the same tab still sees their name and email pre-filled. Clean form when neither source has data.

## Prereqs

- [02-02 Rendered roadmap page](./02-rendered-roadmap-page.md) (the redirect target after submit).
- [00-02 Make.com webhook client](../00-foundations/02-make-webhook-client.md) (`postToMake` + `FthbLm2Payload`).

## Goal

`/orlando-homebuying-roadmap/get.astro` renders an opt-in form. On submit:

1. Validates first_name + email + zip + consent.
2. Builds the `FthbLm2Payload` with `magnet: 'fthb_lm2'` and the right `source` (`'fthb_lm1_tier_b'` or `'fthb_lm2_standalone'`).
3. Fires `postToMake(payload)` (fire-and-forget).
4. Redirects to `/orlando-homebuying-roadmap/view`.

Pre-fill `first_name` and `email` from URL query params if present. Capture `?src=` (or default to `'fthb_lm2_standalone'`).

## Files to create

```
src/pages/orlando-homebuying-roadmap/get.astro
src/scripts/lm2-optin.ts   (the vanilla-JS submit handler)
```

## Fields

- `first_name` (required, text)
- `email` (required, type="email")
- `zip` (required, 5-digit pattern `^\d{5}$`)
- `consent` (required, checkbox): **"I'm OK with [Agent Name] emailing me the roadmap and occasional follow-ups."** Use this exact wording. Replace `[Agent Name]` with the agent's real name when wiring this up (lives in `src/config/agent.ts` if you've already centralized it for the result page footer).

The form `fetch`-POSTs to the **same Make.com webhook URL** as LM1, with `magnet: "fthb_lm2"` distinguishing the routing. Redirect to `/view` immediately without awaiting — same fire-and-forget pattern as LM1.

## LM2 payload schema (passed to `postToMake`)

```json
{
  "magnet": "fthb_lm2",
  "submitted_at": "2026-05-14T18:45:00Z",
  "contact": {
    "first_name": "Carlos",
    "email": "carlos@example.com",
    "zip": "32750"
  },
  "source": "fthb_lm1_tier_b",
  "fthb_lm1_tier": "NINETY_DAY",
  "consent": true,
  "utm_source": "instagram_reel_step_2"
}
```

- `magnet` is the literal string `"fthb_lm2"`.
- `source` is `"fthb_lm1_tier_b"` if `?src=fthb_lm1_tier_b` is in the URL; otherwise `"fthb_lm2_standalone"`. Whitelist — ignore any other `?src=` value (treat as `"fthb_lm2_standalone"`).
- `fthb_lm1_tier` is `null` for `"fthb_lm2_standalone"`. For `"fthb_lm1_tier_b"`, the tier is **not** in the result-page URL params (privacy decision in LM1 task 09), so leave it `null`. Pipedrive looks up the contact by email and already has `fthb_lm1_tier` set from the LM1 submission.
- `consent` is always `true` at submission (the form gates submission on the checkbox).
- `utm_source` is `new URLSearchParams(location.search).get('utm_source')` or `null`.

## What Make.com does with this payload (context, not implemented in this task)

For `source = "fthb_lm1_tier_b"`: Make.com looks up the contact by email, sets `fthb_received_lm2 = true` on the existing Person. A Pipedrive Workflow Automation unenrolls them from the LM1 Tier B campaign and enrolls them in the LM2 transactional + nurture campaign. The "no double email storm" rule lives in that automation, not in this task.

For `source = "fthb_lm2_standalone"`: Make.com creates or updates a Person, sets `fthb_received_lm2 = true`, `fthb_lm2_source = "fthb_lm2_standalone"`. A Pipedrive automation enrolls them in the LM2 transactional + nurture campaign with no LM1 pre-context.

## Pre-fill from URL params and `sessionStorage`

URL params win; `sessionStorage` is the fallback for users who land here in the same tab after completing the LM1 quiz.

```ts
const url = new URL(location.href);
const nParam = url.searchParams.get('n');
const eParam = url.searchParams.get('e');
const srcParam = url.searchParams.get('src');

const ssName = sessionStorage.getItem('fthb_prefill_first_name');
const ssEmail = sessionStorage.getItem('fthb_prefill_email');
```

- `first_name`: use `decodeURIComponent(nParam)` if present, else `ssName`, else leave empty.
- `email`: use `decodeURIComponent(eParam)` if present, else `ssEmail`, else leave empty. (Tier B doesn't include the email in the URL — see [01-09](../01-lm1-readiness-filter/09-result-tier-content.md) — so this is the path that uses `sessionStorage` in practice.)
- `source` for the payload: `srcParam === 'fthb_lm1_tier_b' ? 'fthb_lm1_tier_b' : 'fthb_lm2_standalone'`. Whitelist; ignore any other value.

The keys `fthb_prefill_first_name` and `fthb_prefill_email` are the contract with the LM1 submit handler (see [01-06](../01-lm1-readiness-filter/06-quiz-email-gate-and-submit.md)). Don't read any other `sessionStorage` keys here.

### Visual cue for pre-filled state

If either the URL params or `sessionStorage` provided a name + email, show a small note above the form: **"We have your name and email from the scorecard — just confirm and you're in."** The agent's voice; not a banner ad. If only one of the two fields pre-filled, don't show the note (the user will still see what was filled).

## Submit handler (`src/scripts/lm2-optin.ts`)

1. Prevent default form submit.
2. Validate; show inline errors; focus the first invalid field.
3. Build the payload per the schema above. `fthb_lm1_tier` is always `null` (the result page didn't pass it).
4. Fire `track('fthb_roadmap_optin_submit', { source: payload.source })`.
5. `postToMake(payload)` — **don't await**.
6. `window.location.assign('/orlando-homebuying-roadmap/view')`.

## Other analytics events this task wires

- `fthb_roadmap_optin_view` — fires on page load. Props: `{ source: 'fthb_lm1_tier_b' | 'fthb_lm2_standalone' }` (use the resolved value, same logic as the payload).

## Implementation notes

- **Don't `await` the webhook.** Same fire-and-forget pattern as LM1.
- **No double-submit.** Disable the submit button on click; re-enable after 1 second if the redirect somehow hasn't happened.
- **Mobile-first.** Tap targets ≥ 44px; single column; iOS Safari + Android Chrome.

## Things NOT to do

- Don't add a language field. Bilingual is out for the initial release.
- Don't add a "phone number" field — LM2's opt-in is intentionally lighter than LM1's email gate.
- Don't gate the roadmap read on this form. The opt-in enrolls them in nurture; the roadmap at `/view` is open access.
- Don't try to pre-fill from `localStorage` or a cookie. The only allowed prefill sources are URL params and the two `sessionStorage` keys named above.
- Don't try to detect "is this contact already in Pipedrive" client-side. Make.com does that lookup; the form doesn't need to know.

## Definition of Done

- [ ] `/orlando-homebuying-roadmap/get` renders the form
- [ ] Visiting with `?n=Maria&src=fthb_lm1_tier_b` pre-fills the name and shows the "we have your name and email from the scorecard" note
- [ ] Visiting with no URL params but with `fthb_prefill_first_name` + `fthb_prefill_email` in `sessionStorage` pre-fills both fields and shows the note
- [ ] Visiting with no URL params and empty `sessionStorage` shows a clean form
- [ ] Invalid form shows inline errors and prevents submit
- [ ] Valid form submission fires the Make.com webhook (verify in execution log) and redirects to `/view`
- [ ] Blocking the webhook URL in DevTools Network doesn't prevent the redirect
- [ ] Payload has `magnet: 'fthb_lm2'` and the correct `source` value based on `?src=`
- [ ] `fthb_roadmap_optin_view` and `fthb_roadmap_optin_submit` analytics events fire with `source`

## Verification

```
http://localhost:4321/orlando-homebuying-roadmap/get
http://localhost:4321/orlando-homebuying-roadmap/get?n=Maria&src=fthb_lm1_tier_b
```

Submit each. Confirm:
- Clean form (no URL params, `sessionStorage` empty) → payload has `source: 'fthb_lm2_standalone'`, name from the form input
- Pre-filled from URL → payload has `source: 'fthb_lm1_tier_b'`, name from the URL param

Then end-to-end the LM1 → LM2 bridge: take the LM1 quiz, submit with a Tier B answer set, click through to the Roadmap opt-in from the result page in the same tab. Confirm both name and email are pre-filled from `sessionStorage` and the prefill note is showing.
