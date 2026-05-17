Build the static landing page that promotes the readiness scorecard. Hero, sub-hero bullets, FAQ. One CTA, sending the visitor into the quiz.

## Goal

`/orlando-homebuying-readiness-quiz/index.astro` renders a mobile-first page with the hero (locked title + subhead), the 3 "what you'll get" bullets, the FAQ accordion, and a single primary CTA that links to `/orlando-homebuying-readiness-quiz/start`.

## Files to create

```
src/pages/orlando-homebuying-readiness-quiz/index.astro
```

## Locked copy (ship verbatim)

The title and subhead are locked — do not paraphrase on landing pages, ads, or DMs. Copy/paste them.

### Hero

> **The Orlando First-Time Buyer Readiness Score**
> Know in 7 Minutes If You're 30, 90, or 180 Days Away From Your First Home.
>
> Answer 10 plain-English questions and get a personalized readiness score, with an exact timeline and the 2–3 specific things standing between you and your first Orlando home. No credit pull. No phone call. No "we'll be in touch."
>
> [ Start the 7-minute scorecard ]
>
> _Free. Built by a licensed Orlando agent._

### Sub-hero. "What you'll get" (3 bullets, not a wall)

- A readiness tier. Ready Now, 90-Day Sprint, or Foundation Phase
- The 2 specific mistakes buyers in your tier make most often (so you don't)
- A 1-page Orlando market snapshot. What $475K actually buys along the I-4 corridor right now, with examples from Sanford and Lake Mary on the Seminole side and Winter Park and College Park on the Orange side

### FAQ (below the fold)

- _Will you pull my credit?_ No. You self-report a range. We can't see anything.
- _Is this an ad to get me on the phone?_ No. The result is yours. If you want to talk, there's a free strategy session at the end. Only if you want it.
- _Why 10 questions?_ That's the minimum to give you a real answer. Anything less is a brochure.

## Implementation notes

- **Locked copy.** The headline (`The Orlando First-Time Buyer Readiness Score`) and subhead (`Know in 7 Minutes If You're 30, 90, or 180 Days Away From Your First Home.`) must be exact. Same for the 3 sub-hero bullets and every FAQ question/answer above.
- Wrap in `<BaseLayout title="The Orlando First-Time Buyer Readiness Score" description="...">` (or whatever prop names `EXISTING-SITE-NOTES.md` Section 3 documents). Description should be 140–160 chars and capture the offer.
- Above the fold: hero only (no nav, no logo overload). The CTA button is the entire interaction.
- Below the fold: the 3-bullet "what you'll get," then the FAQ.
- **FAQ as a `<details>` accordion.** Pure HTML; no JS. Each FAQ item is one `<details><summary>...question...</summary><p>...answer...</p></details>`. Tailwind `prose` plus a small `[&_summary]:cursor-pointer` selector is enough styling (if the host site uses Tailwind; otherwise adapt per `EXISTING-SITE-NOTES.md` Section 5).
- CTA button is a real `<a href="/orlando-homebuying-readiness-quiz/start">`. Style it as a button; don't use an actual `<button>` for navigation.
- **Mobile-first.** iOS Safari + Android Chrome on phone widths (375px) is the canonical experience. Tap targets ≥ 44px. Single column. Desktop falls out of clean responsive scale-up; do not invest in desktop-specific polish.
- Fire the `fthb_readiness_landing_view` analytics event on page load (call the site's `track()` helper from `EXISTING-SITE-NOTES.md` Section 4; once on `DOMContentLoaded`). If the analytics provider auto-fires page views, also fire this custom event manually for parity with the rest of the funnel events.
- Fire `fthb_readiness_quiz_start` when the CTA is clicked. Add a tiny inline `<script>` that listens for the click and calls `track()` before navigation. Use `pointerdown` so the event fires even on fast nav.

## Things NOT to do

- Don't add nav, footer links, social links, agent photo, or anything else above the fold. The hero CTA is the only thing that matters.
- Don't paraphrase the locked copy. If the agent later requests a change, it's a single edit in this file; don't preemptively soften.
- Don't add a phone-number field or "request a call" form. The funnel architecture is intentional.

## Definition of Done

- [ ] Page renders correctly on iOS Safari and Android Chrome at phone widths (375px primary)
- [ ] All copy matches the verbatim text above (diff-check by pasting)
- [ ] CTA navigates to `/orlando-homebuying-readiness-quiz/start` (which won't exist yet — that's task 04; for now the link 404 is fine)
- [ ] FAQ items open and close on tap and on keyboard (Enter) — the native `<details>` element handles this
- [ ] `fthb_readiness_landing_view` fires on page load; `fthb_readiness_quiz_start` fires on CTA click
- [ ] Lighthouse mobile score ≥ 95 for Performance and Accessibility (no scripts beyond analytics and the tiny event listener; everything else is static HTML)

## Verification

Visit `http://localhost:4321/orlando-homebuying-readiness-quiz` in mobile DevTools view, tab through the page with keyboard, open every FAQ, click the CTA.
