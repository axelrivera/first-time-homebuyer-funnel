
Most events were wired in earlier tasks. This task is the sweep: every LM1 event is firing exactly once at the right moment, with the right props.

## Prereqs

- [01-09 Tier A/B/C content + CTAs](./09-result-tier-content.md) (last LM1 task with new events).
- [00-01 Audit existing site](../00-foundations/01-audit-existing-site.md) (analytics helper signature; the `track()` function the funnel calls).

## Goal

All 7 LM1 events are firing correctly. No duplicates, no missing events, props match below exactly. The agent can open the analytics dashboard (Plausible / PostHog / GA4 — whichever the host site uses) and see a real funnel.

## Event checklist (LOCKED — do not add or remove)

| Event | When | Props | Where (file) |
|---|---|---|---|
| `fthb_readiness_landing_view` | Landing page load | none | `src/pages/orlando-homebuying-readiness-quiz/index.astro` |
| `fthb_readiness_quiz_start` | User clicks "Start" on the landing CTA | none | `src/pages/orlando-homebuying-readiness-quiz/index.astro` (CTA listener) |
| `fthb_readiness_question_answered` | Each Q advances **forward** (not back) | `{ q_id: 'q1_credit_range' \| ..., answer_key: '...' }` | `src/scripts/quiz.ts` (state-machine forward advance) |
| `fthb_readiness_email_gate_shown` | Email gate becomes the active step | none | `src/scripts/quiz.ts` (when `currentStep` reaches the email-gate step) |
| `fthb_readiness_email_gate_submit` | Form submits after passing validation | `{ tier: 'A' \| 'B' \| 'C' }` | `src/scripts/quiz.ts` (submit handler, before redirect) |
| `fthb_readiness_result_view` | Result page load (real, not `?preview=`) | `{ tier: 'A' \| 'B' \| 'C', score: number }` | `src/pages/orlando-homebuying-readiness-quiz/result.astro` (client `<script>` block) |
| `fthb_readiness_cta_click` | CTA on result page clicked | `{ tier: 'A' \| 'B' \| 'C', cta_target: 'bss' \| 'fthb_lm2' \| 'none' }` | `src/components/result/Tier*Content.astro` |

The exact `track()` signature comes from `EXISTING-SITE-NOTES.md` Section 4. This task adapts each call to that signature; the event names and props above don't change.

## Implementation notes

- **No duplicates.** Analytics providers' auto page-view (Plausible, GA4) is separate from these manual events. If you fire `fthb_readiness_landing_view` manually, the dashboard will show *both* an auto page-view *and* the custom event for the landing page. That's fine — they answer different questions. Just don't fire the same custom event twice from the same load.
- **`fthb_readiness_question_answered` only on forward advance.** Going Back to Q3 from Q4, then forward again, **should** re-fire the Q3 event (because forward-advance fired). What it should not do is fire on Back itself. The state machine knows whether `currentStep` increased; gate the fire on that.
- **`fthb_readiness_result_view` skips preview mode.** If `?preview=` is set, do not fire — preview is for the agent, not real funnel data.
- **CTA tracking on `pointerdown`.** Click events sometimes get cancelled by navigation in mobile browsers; `pointerdown` fires earlier and is more reliable. Pair with the link so the event is sent before the browser navigates away. If the host site's `track()` wrapper supports `keepalive`, prefer that.
- **No PII in event props.** Tier + score + answer keys are enough; emails, ZIPs, and full names belong in Pipedrive, not in analytics props.
- **No relay through Make.com.** Analytics is direct-from-browser. If Make.com is down, analytics must still work.

## Things NOT to do

- Don't add new events beyond the table above. Add them in Phase 4 when there's a specific question to answer.
- Don't include emails, ZIPs, or full names in event props.
- Don't relay events through Make.com. Direct-from-browser only.

## Definition of Done

- [ ] Each row in the table above maps to exactly one call site in the codebase
- [ ] A full end-to-end take of the quiz produces this sequence in the analytics Realtime view:
  1. `fthb_readiness_landing_view`
  2. `fthb_readiness_quiz_start`
  3. `fthb_readiness_question_answered` × 10 (one per question, with correct `q_id` / `answer_key` for each)
  4. `fthb_readiness_email_gate_shown`
  5. `fthb_readiness_email_gate_submit` (with `tier`)
  6. `fthb_readiness_result_view` (with `tier`, `score`)
  7. `fthb_readiness_cta_click` (when the result CTA is clicked, for Tier A/B; not expected on Tier C)
- [ ] Going Back from Q5 to Q3 and then re-advancing forward re-fires Q3/Q4/Q5 events on the forward path; Back itself fires nothing.
- [ ] `?preview=` URLs do not fire `fthb_readiness_result_view`

## Verification

- Take the quiz top-to-bottom; watch the analytics Realtime view.
- Take the quiz, go back from Q5 to Q3, change Q3, advance forward — the Q3, Q4, Q5 events should each fire exactly twice (once originally, once after the backtrack). The decision encoded here: fire **only on forward advance** (Back fires nothing); the re-fire on re-advance is expected and acceptable. Document the choice in a comment in `quiz.ts`.
