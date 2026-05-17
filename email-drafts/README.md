# `email-drafts/` — working layer for FTHB funnel emails

This directory is your **active editing surface** for every email in the FTHB funnel. One file per email, with frontmatter status fields so you can see where each one stands at a glance.

> **Relationship to `docs/`:** the email files in [`docs/04-readiness-filter-emails.md`](../docs/04-readiness-filter-emails.md) and [`docs/07-process-map-emails.md`](../docs/07-process-map-emails.md) are the **spec snapshot**. This folder is where polish happens. When an email graduates to `status: POLISHED`, copy the locked version back into the relevant docs file (or leave a pointer) so the spec stays in sync.

## Status legend (frontmatter `status:` field)

| Status     | Meaning |
|------------|---------|
| `DRAFT`    | Initial copy from the spec; not polished yet. |
| `REVIEWING`| You're actively editing or showing it to a second pair of eyes. |
| `POLISHED` | Final copy. Ready to paste into Pipedrive Campaigns. |
| `SHIPPED`  | Live in Pipedrive and sending. Don't edit here without re-syncing. |
| `BACKLOG`  | Topic queued but no draft body yet (only used in C3+). |
| `TEMPLATE` | A pattern file (e.g., monthly market update), not a single send. |

## Full status tracker

### LM1 — *The Orlando First-Time Buyer Readiness Score*

| File | ID  | Day | Subject | Status |
|---|---|---|---|---|
| [00-transactional-all-tiers.md](lm1-readiness-filter/00-transactional-all-tiers.md) | LM1-E0 | 0 | Your Orlando readiness score: {{tier_label}} | DRAFT |
| **Tier A — Ready Now** | | | | |
| [A1-day-00-ready-now-meaning.md](lm1-readiness-filter/tier-a-ready-now/A1-day-00-ready-now-meaning.md) | LM1-A1 | 0 | One thing about your "Ready Now" score | DRAFT |
| [A2-day-02-seminole-vs-orange.md](lm1-readiness-filter/tier-a-ready-now/A2-day-02-seminole-vs-orange.md) | LM1-A2 | 2 | Seminole County vs. Orange County: a 90-second read | DRAFT |
| [A3-day-05-pre-approval-question.md](lm1-readiness-filter/tier-a-ready-now/A3-day-05-pre-approval-question.md) | LM1-A3 | 5 | The pre-approval question I always ask first | DRAFT |
| [A4-day-09-inspection-contingency.md](lm1-readiness-filter/tier-a-ready-now/A4-day-09-inspection-contingency.md) | LM1-A4 | 9 | The one contingency that does the heavy lifting | DRAFT |
| [A5-day-14-last-note.md](lm1-readiness-filter/tier-a-ready-now/A5-day-14-last-note.md) | LM1-A5 | 14 | Last note from me | DRAFT |
| **Tier B — 90-Day Sprint** | | | | |
| [B1-day-00-game-plan.md](lm1-readiness-filter/tier-b-ninety-day-sprint/B1-day-00-game-plan.md) | LM1-B1 | 0 | Your 90-day game plan: start here | DRAFT |
| [B2-day-02-before-house-shopping.md](lm1-readiness-filter/tier-b-ninety-day-sprint/B2-day-02-before-house-shopping.md) | LM1-B2 | 2 | Before you look at a single house | DRAFT |
| [B3-day-04-builders-lender.md](lm1-readiness-filter/tier-b-ninety-day-sprint/B3-day-04-builders-lender.md) | LM1-B3 | 4 | When to use the builder's lender | DRAFT |
| [B4-day-07-where-stuck.md](lm1-readiness-filter/tier-b-ninety-day-sprint/B4-day-07-where-stuck.md) | LM1-B4 | 7 | Quick question for you | DRAFT |
| [B5-day-12-sprint-graduation.md](lm1-readiness-filter/tier-b-ninety-day-sprint/B5-day-12-sprint-graduation.md) | LM1-B5 | 12 | When the Sprint becomes Ready Now | DRAFT |
| [B6-day-21-last-note.md](lm1-readiness-filter/tier-b-ninety-day-sprint/B6-day-21-last-note.md) | LM1-B6 | 21 | Last note (then I back off) | DRAFT |
| **Tier C — Foundation Phase** | | | | |
| [C0-day-00-welcome.md](lm1-readiness-filter/tier-c-foundation/C0-day-00-welcome.md) | LM1-C0 | 0 | What "Foundation Phase" actually means | DRAFT |
| [C1-day-14-credit.md](lm1-readiness-filter/tier-c-foundation/C1-day-14-credit.md) | LM1-C1 | 14 | The one place to actually pull your credit (and why) | DRAFT |
| [C2-day-28-savings.md](lm1-readiness-filter/tier-c-foundation/C2-day-28-savings.md) | LM1-C2 | 28 | Where to actually park your home fund | DRAFT |
| [C3-plus-rotating-topics.md](lm1-readiness-filter/tier-c-foundation/C3-plus-rotating-topics.md) | LM1-C3+ | 42+ | 10-topic bi-weekly rotation (backlog) | BACKLOG |
| **Shared** | | | | |
| [_shared/monthly-market-update.md](lm1-readiness-filter/_shared/monthly-market-update.md) | SHARED-MONTHLY | monthly | (monthly market update template) | TEMPLATE |

### LM2 — *The 9-Step First Home Roadmap*

| File | ID  | Day | Subject | Status |
|---|---|---|---|---|
| [00-transactional.md](lm2-process-map/00-transactional.md) | LM2-E0 | 0 | Your 9-Step Orlando Home Roadmap (link + PDF inside) | DRAFT |
| [nurture/N1-day-03-which-step.md](lm2-process-map/nurture/N1-day-03-which-step.md) | LM2-N1 | 3 | Which step did you start on? | DRAFT |
| [nurture/N2-day-07-step-3-to-4.md](lm2-process-map/nurture/N2-day-07-step-3-to-4.md) | LM2-N2 | 7 | The Step 3 → Step 4 jump | DRAFT |
| [nurture/N3-day-14-roadmap-to-conversation.md](lm2-process-map/nurture/N3-day-14-roadmap-to-conversation.md) | LM2-N3 | 14 | When the roadmap turns into a conversation | DRAFT |

## Per-magnet indexes

- [LM1 index](lm1-readiness-filter/_index.md) — campaign list, file layout, tier-routing rules
- [LM2 index](lm2-process-map/_index.md) — single-campaign layout, Tier B email-storm rule

## Editing workflow

1. Open one file at a time. Read the frontmatter and the **Polish notes** checklist at the bottom.
2. Edit the body inside the fenced code block. Keep merge fields in `{{double_braces}}` form so Pipedrive can resolve them as-is.
3. Update `status:` as you progress (`DRAFT` → `REVIEWING` → `POLISHED`).
4. Bump `last_edited:` to today's date.
5. Update **this README**'s status table so the at-a-glance view stays accurate. (You can grep `status:` across the tree if you'd rather automate it.)
6. When you flip to `SHIPPED`, paste the final body into Pipedrive Campaigns and mirror the polished version back into the matching block in `docs/04-readiness-filter-emails.md` or `docs/07-process-map-emails.md`.

## Quick status grep

```bash
# from this directory
grep -rh '^status:' . | sort | uniq -c
```

Counts how many emails are in each status state — useful when you want a one-line "where am I."

## What lives elsewhere

- **Behavior/scoring/routing specs** → `../docs/` (source of truth for *what the funnel does*).
- **Build tasks** → `../requirements/` (Claude Code build plan; treats `docs/` as truth).
- **Host-site bindings** → `../requirements/EXISTING-SITE-NOTES.md` once the audit task ships.
