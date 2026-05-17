
Build the standalone landing page that promotes the 9-Step Roadmap from content (e.g., an Instagram reel about one step). Hero, sub-hero, "who it's for," primary CTA, and a secondary CTA back to the LM1 scorecard.

## Prereqs

- All of LM1 (01–10). LM2 begins after LM1 ships, since the Tier B result page CTA assumes `/orlando-homebuying-roadmap/get` exists from task 03 below.

## LM2 has exactly TWO entry points (locked architecture)

LM2 is **only** promoted from:

1. The LM1 Tier B result page CTA (entry point #1).
2. This standalone landing page at `/orlando-homebuying-roadmap` (entry point #2), used for content specifically about the buying *process* (e.g., a reel walking through one step).

LM2 is **not** linked from:

- The main agent homepage navigation.
- Tier A or Tier C result pages (Tier A goes to BSS, Tier C goes to nurture only).
- The Instagram/Facebook bio (LM1 owns that slot).

Adding a third entry point fragments the segmentation the funnel was built on. Do not add one.

## Goal

`/orlando-homebuying-roadmap/index.astro` renders the static landing page. Primary CTA → `/orlando-homebuying-roadmap/get`. Secondary CTA → `/orlando-homebuying-readiness-quiz` (the LM1 scorecard, for prospects who haven't taken it).

## Files to create

```
src/pages/orlando-homebuying-roadmap/index.astro
```

## Locked copy (ship verbatim)

### Hero

> **The 9-Step First Home Roadmap**
> Exactly What Happens Between "I Think I'm Ready" and Keys in Your Hand in Orlando.
>
> A step-by-step visual map of every milestone, decision, and potential mistake in the Orlando home-buying process, so you walk in knowing more than most buyers who've done this before.
>
> [ Send me the roadmap (free) → ]
>
> *Delivered to your inbox in 60 seconds. No phone call required.*

### Sub-hero: "What's in it"

- The 9 Steps from "Check Your Credit" to "Close Day": what *you* do, what *your agent* does, how long each takes
- The 3 places first-time buyers in Orlando lose money (named, specific, avoidable)
- Sanford to Downtown Orlando gotchas: HOA timelines, rainy-season inspection issues, Seminole-vs.-Orange school-zone resale math, the commute trade-off between Sanford and Downtown Orlando
- The Pre-Approval Cheat Sheet: documents to gather + the one credit number that matters more than your score

### Sub-hero: Who it's for

> If you took my Readiness Scorecard and landed in the 90-Day Sprint tier, this is the document I built specifically for you. If you didn't take the scorecard, you can take it later, but the roadmap is useful to anyone in the first 6 months of the home-buying process in Orlando.
>
> [ Or, take the 7-minute Readiness Scorecard first → ]

## Implementation notes

- **Locked copy:** title (`The 9-Step First Home Roadmap`) and subhead (`Exactly What Happens Between "I Think I'm Ready" and Keys in Your Hand in Orlando.`) verbatim. Don't paraphrase.
- Wrap in `<BaseLayout title="The 9-Step First Home Roadmap" description="...">` (props per `EXISTING-SITE-NOTES.md` Section 3).
- Hero pattern matches LM1: above the fold = hero + single primary CTA. Below = "what's in it" 4 bullets + "who it's for" + the secondary "take the scorecard first" CTA.
- The secondary CTA is a quieter style (text link or outlined button) so it doesn't compete with the primary opt-in CTA.
- **Fire `fthb_roadmap_landing_view`** on page load (only fires from this standalone entry — the Tier B path lands directly on `/get`, skipping this page).
- Mobile-first; iOS Safari + Android Chrome at 375px is the canonical experience.

## Things NOT to do

- Don't link this page from the agent's homepage nav, Instagram bio, or anywhere else. LM2 has exactly two entry points. See the architecture note above.
- Don't pre-pitch the BSS here. The CTA is the roadmap opt-in, not the strategy session.
- Don't show the agent's photo, bio, or testimonials above the fold. The offer is the focus.

## Definition of Done

- [ ] Page renders on iOS Safari + Android Chrome at phone widths
- [ ] Locked copy matches the text above verbatim
- [ ] Primary CTA navigates to `/orlando-homebuying-roadmap/get` (won't exist until task 03 — 404 OK for now)
- [ ] Secondary CTA navigates to `/orlando-homebuying-readiness-quiz`
- [ ] `fthb_roadmap_landing_view` analytics event fires on page load
- [ ] Lighthouse mobile score ≥ 95 for Performance + Accessibility

## Verification

`http://localhost:4321/orlando-homebuying-roadmap` — review copy diff-style against the text above, click both CTAs.
