
This directory breaks the funnel build into small, sequential, **atomic** tasks. Each file is one Claude Code session — read it, build it, verify the Definition of Done, move on.

## Where the code lives

**The funnel is built into the existing axelrivera.com Astro site, not a new project in this repo.** This repo holds the build tasks; the code lands in the existing site's repo.

The original task drafts assumed a greenfield Astro project. They don't — the existing site already has `BaseLayout`, analytics, Tailwind (or its equivalent), folder conventions, hosting, and deploy. The funnel adapts to the existing site, not the other way around.

[00-foundations/01-audit-existing-site.md](./00-foundations/01-audit-existing-site.md) produces **[EXISTING-SITE-NOTES.md](./EXISTING-SITE-NOTES.md)** — a stable reference file documenting the existing site's `BaseLayout`, analytics helper, Tailwind config (if any), folder conventions, and routing. Every downstream task implicitly defers to that notes file for project-specific bindings. **Read it before starting any LM1 or LM2 task.**

## How to use this directory

- **Files are numbered.** Do them in order within each subdirectory. Cross-subdirectory order is below.
- **Each task is atomic.** The locked copy, payload schemas, scoring math, stable enum keys, question text, tier thresholds, and analytics event tables are inlined into the task that needs them. You can build a task without opening anything else in this repo (except `EXISTING-SITE-NOTES.md` for host-site bindings — see below).
- **Each task has a Definition of Done.** If the DoD isn't met, the task isn't finished — even if "the code looks right."
- **Prereqs are explicit.** Don't start a task whose prereqs haven't shipped.
- **Tasks defer to [EXISTING-SITE-NOTES.md](./EXISTING-SITE-NOTES.md) for project-specific patterns.** Where a task says "wrap in `<BaseLayout title=...>`" or "fire `track(...)`" or "use Tailwind class `...`," interpret that against the existing site's actual `BaseLayout` props, analytics helper signature, and styling approach. The notes file is the source of truth for those host-site bindings; the task files describe the funnel-specific intent.
- **Out of scope for these files:** Pipedrive Campaigns setup, Pipedrive Workflow Automations, Make.com scenario configuration, Google Sheet wiring. The Astro side only POSTs to the Make.com webhook; the rest happens off-site and is operational, not a build task.

## What's in [../docs/](../docs/) vs. what's in here

- **[../docs/](../docs/)** — the strategic source-of-truth specs and content drafts. Read once at project start to understand *why* the funnel is shaped the way it is. After that, you don't need to open them to build any individual task — the task files have the locked copy, payload schemas, and scoring rules you need inlined.
- **[requirements/](.)** — the build tasks. Each one is self-contained for execution. If a behavior or copy question comes up that the task doesn't answer, *then* fall back to docs/. In practice that should be rare.

## Overall order

1. **Finish [00-foundations/](./00-foundations/) first.** The audit task produces `EXISTING-SITE-NOTES.md`, which every downstream task depends on. The webhook-client task wires the only piece of new infrastructure the funnel needs.
2. **Then build LM1 in order** ([01-lm1-readiness-filter/](./01-lm1-readiness-filter/), tasks 01 through 10). Top of the funnel; ship it before LM2 even starts.
3. **Then build LM2 in order** ([02-lm2-process-map/](./02-lm2-process-map/), tasks 01 through 05). LM2 depends on LM1's Tier B result page CTA existing — don't start it until LM1's [task 09](./01-lm1-readiness-filter/09-result-tier-content.md) is done.

## Directory map

### [00-foundations/](./00-foundations/)
Audit the existing axelrivera.com site, then add the only missing piece (the Make.com webhook client). Everything else the funnel needs — Astro scaffold, base layout, analytics, Tailwind, hosting — is already in place on the existing site.

| # | Task | Builds |
|---|---|---|
| 01 | [Audit existing site](./00-foundations/01-audit-existing-site.md) | `requirements/EXISTING-SITE-NOTES.md` (read-only audit; produces the reference doc) |
| 02 | [Make.com webhook client](./00-foundations/02-make-webhook-client.md) | One TS module (path per audit notes) with `postToMake` + typed `FthbLm1Payload` / `FthbLm2Payload` |

### [01-lm1-readiness-filter/](./01-lm1-readiness-filter/)
The Readiness Score funnel. Top of the funnel, top of the priority list. Each task contains the locked copy, stable enum keys, scoring math, or analytics event definitions it needs.

| # | Task | Builds |
|---|---|---|
| 01 | [Landing page](./01-lm1-readiness-filter/01-landing-page.md) | `/orlando-homebuying-readiness-quiz/index.astro` (hero + 3 bullets + FAQ) |
| 02 | [Scoring config module](./01-lm1-readiness-filter/02-config-module.md) | `src/config/readiness.ts` (10 question point maps + thresholds + 4 overrides) |
| 03 | [Scoring engine](./01-lm1-readiness-filter/03-scoring-engine.md) | `src/lib/scoring.ts` (pure `computeScore` function) |
| 04 | [Quiz shell + state machine + Q1](./01-lm1-readiness-filter/04-quiz-shell-and-state-machine.md) | `/orlando-homebuying-readiness-quiz/start.astro` |
| 05 | [Wire up Q2–Q10](./01-lm1-readiness-filter/05-quiz-all-10-questions.md) | All 10 questions visible in the shell |
| 06 | [Email gate + submit handler](./01-lm1-readiness-filter/06-quiz-email-gate-and-submit.md) | Final screen + `postToMake` + redirect |
| 07 | [Market snapshot component](./01-lm1-readiness-filter/07-market-snapshot-component.md) | `src/components/MarketSnapshot.astro` + `src/data/market-snapshot.json` |
| 08 | [Result page shell + preview mode](./01-lm1-readiness-filter/08-result-page-shell.md) | `/orlando-homebuying-readiness-quiz/result.astro` |
| 09 | [Tier A/B/C content](./01-lm1-readiness-filter/09-result-tier-content.md) | Long-form copy + CTAs on the result page |
| 10 | [Analytics events](./01-lm1-readiness-filter/10-analytics-events.md) | All 7 LM1 events firing |

### [02-lm2-process-map/](./02-lm2-process-map/)
The 9-Step Roadmap funnel. Build only after LM1 task 09 lands (the Tier B result CTA needs `/orlando-homebuying-roadmap/get` to exist).

| # | Task | Builds |
|---|---|---|
| 01 | [Landing page](./02-lm2-process-map/01-landing-page.md) | `/orlando-homebuying-roadmap/index.astro` |
| 02 | [Rendered roadmap page](./02-lm2-process-map/02-rendered-roadmap-page.md) | `/orlando-homebuying-roadmap/view.astro` (all 9 steps + gotchas + money losses + cheat sheet) |
| 03 | [Opt-in form + submit](./02-lm2-process-map/03-opt-in-form.md) | `/orlando-homebuying-roadmap/get.astro` |
| 04 | [Static PDF asset](./02-lm2-process-map/04-static-pdf-asset.md) | `public/assets/orlando-9-step-roadmap.pdf` |
| 05 | [Analytics events](./02-lm2-process-map/05-analytics-events.md) | All 6 LM2 events firing |

## What "done" means at each milestone

- **Foundations done:** `EXISTING-SITE-NOTES.md` exists and has a concrete answer for every section. `postToMake({...})` POSTs to a real Make.com webhook from a scratch page on the existing site. The funnel routes (`/orlando-homebuying-readiness-quiz`, `/orlando-homebuying-roadmap`) are confirmed unclaimed by the existing site.
- **LM1 done:** A friend takes the quiz on their phone, gets the right result page, and a Pipedrive Person is created with the right tier within 60 seconds.
- **LM2 done:** A Tier B contact clicks the LM2 CTA, sees the roadmap, the static PDF is downloadable, and the LM1 Tier B campaign is unenrolled in Pipedrive.
- **Funnel shipped:** All DoDs above. Phase 3 (monthly nurture + analytics dashboards) and Phase 4 (iteration) are tracked elsewhere, not here.
