
Build the one-page quiz: a single Astro page with vanilla-JS in-memory state. This task ships the page shell, the question-advance state machine, and Q1 fully wired as a proof-of-pattern. Q2–Q10 land in the next task.

## Why this is ONE page (not multiple routes)

The site has no cookies, no `localStorage`, no env vars by design, and `sessionStorage` is only used after a successful submit (to carry `first_name` + `email` to the LM2 Roadmap opt-in form — see task [06](./06-quiz-email-gate-and-submit.md)). In-flight quiz state is never persisted anywhere. Without in-flight storage, separate routes would lose state between Q10 and the email gate. One page, one in-memory state machine. Reloading restarts the quiz — that's intentional. If completion suffers in Phase 4, the iteration is to **shorten the quiz**, not add storage. In-flight storage adds compliance surface area we don't want.

## Goal

`/orlando-homebuying-readiness-quiz/start.astro` is one page. On load, only Q1 is visible. Answering Q1 advances to Q2 (which is empty for now). A back button returns to Q1 with the answer still selected. No page navigation; pure DOM show/hide. State lives in a JS variable; reloading the page wipes it.

## Files to create

```
src/pages/orlando-homebuying-readiness-quiz/start.astro
src/components/QuizQuestion.astro   (the radio-group component used per question)
src/scripts/quiz.ts                 (the vanilla-JS state machine; imported as a script)
```

(Adjust paths to whatever `EXISTING-SITE-NOTES.md` Section 2 documents as the convention for components and client scripts.)

## Q1 content (the proof-of-pattern question)

**Prompt:**
> **What's your best guess at your current credit score?**
> If you're not sure, pick the range that feels right. We're not pulling anything.

**Options** (stable enum key → UI label, in this order):

- `740_plus` → "740 or higher"
- `680_739` → "680 – 739"
- `620_679` → "620 – 679"
- `580_619` → "580 – 619"
- `below_580` → "Below 580"
- `unknown` → "I have no idea"

Each option is a real `<input type="radio" name="q1_credit_range" value="...">`. The `value` attribute is the stable enum key.

## Implementation notes

- **One page, eleven screens.** The 10 questions plus the email gate are all rendered into the same Astro page. Each screen is a `<section data-step="1" hidden>` (or similar). The state machine toggles the `hidden` attribute as the user advances. This task only wires up Q1 and a stub Q2 — the rest comes in [05](./05-quiz-all-10-questions.md) and [06](./06-quiz-email-gate-and-submit.md).
- **State machine** (`src/scripts/quiz.ts`):
  - Holds `answers: Partial<Answers>` and `currentStep: number` in module-scope variables.
  - Exposes `goTo(step: number)` that updates `currentStep`, toggles `hidden` on all step sections, scrolls to top, updates the progress bar, and fires `track('fthb_readiness_question_answered', {...})` when advancing **forward** from an answered step (not on Back).
  - Handles Back: re-shows previous step; the selected radio button is still checked because the DOM never re-rendered.
  - On Enter or click of the "Next" button: read the active radio in the current step, write it into `answers`, call `goTo(currentStep + 1)`.
- **Per-question component (`QuizQuestion.astro`)** props: `{ step: number; qid: 'q1_credit_range' | ...; label: string; helpText?: string; options: Array<{ key: string; label: string }> }`. Renders a `<fieldset><legend>{label}</legend>...radios...</fieldset>` with a "Back" button (hidden on step 1) and a "Next" button.
- **Progress bar** at the top: `<div aria-live="polite">Question 1 of 10</div>` + a visual bar. Update via state machine.
- **No in-flight storage.** Don't add `localStorage.setItem`, `sessionStorage.setItem`, or cookies inside the state machine. If the user reloads mid-quiz, they restart. This is the design. (The `sessionStorage` write for the LM2 Roadmap prefill bridge happens only in the submit handler in task [06](./06-quiz-email-gate-and-submit.md), after a successful submit — not inside the state machine.)

## Mobile-first accessibility non-negotiables (apply to every device)

- iOS Safari + Android Chrome at phone widths (375px) is the canonical experience.
- Real `<input type="radio">`, real `<label>` per option. Tap targets ≥ 44px.
- Keyboard: Tab cycles options, Space/Enter selects, Enter (after selection) advances. Implement via the form's submit handler.
- One question per screen, single-column layout, mobile-first.
- Progress bar at the top (e.g., "3 of 10"), paired with a numeric label — color is never the only signal.
- Back button on every question. Never lose answers on back.
- Semantic HTML: `<fieldset>`, `<legend>`, `<label>` per option.
- SR-only announcement when step changes: `<span class="sr-only" aria-live="assertive">` updates to "Question 3 of 10" when state changes.

When mobile vs. desktop trade-offs come up, **mobile wins every time**. Desktop just needs to not break.

## Things NOT to do

- Don't split the quiz across multiple Astro routes. The `/start` and `/contact` flow is collapsed into one page on purpose (no in-flight storage means no cross-route state for the quiz answers).
- Don't use Astro server-side state. Everything is rendered statically; interactivity is client-side JS only.
- Don't pull in a state-management library. The state is two variables.
- Don't introduce a framework island (React, Vue, Svelte) for the quiz. Vanilla JS is the policy.

## Definition of Done

- [ ] `/orlando-homebuying-readiness-quiz/start` loads, shows Q1 only, progress bar reads "Question 1 of 10"
- [ ] Selecting an option and clicking Next reveals Q2 (stub for now); progress reads "Question 2 of 10"
- [ ] Clicking Back returns to Q1; the previously selected option is still checked
- [ ] Reloading the page restarts at Q1 with no selection (no storage)
- [ ] Tab order works: through the radios, to Back/Next, in order
- [ ] Pressing Enter after selecting an option advances to the next step
- [ ] A11y inspector (axe, Lighthouse) reports no issues on the Q1 screen
- [ ] `fthb_readiness_question_answered` analytics event fires when advancing from Q1, with props `{ q_id: 'q1_credit_range', answer_key: '...' }`

## Verification

Take the quiz manually in mobile DevTools view. Hit reload at Q2 — should restart from Q1.
