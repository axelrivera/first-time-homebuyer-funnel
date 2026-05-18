# LM1 — *The Orlando First-Time Buyer Readiness Score* (email campaigns)

**Source spec:** [docs/04-readiness-filter-emails.md](../../docs/04-readiness-filter-emails.md)

LM1 is delivered as **one Pipedrive campaign per email** (each email is a standalone campaign in Pipedrive). Names follow the convention `FTHB Readiness Quiz - {Segment} Day {N}`, with `(Email 0)` reserved for the transactional. The shared monthly market update is a single recurring campaign.

| Segment | Tier | Per-email campaigns | Goal |
|---|---|---|---|
| Result Delivery | all | `FTHB Readiness Quiz - Result Delivery (Email 0)` | deliver result link within 60s of submit |
| Ready Now | A (READY_NOW) | `FTHB Readiness Quiz - Ready Now Day {0, 2, 5, 9, 14}` | book BSS |
| 90-Day Sprint | B (NINETY_DAY) | `FTHB Readiness Quiz - 90-Day Sprint Day {0, 2, 4, 7, 12, 21}` | get them into LM2, then BSS |
| Foundation Phase | C (FOUNDATION) | `FTHB Readiness Quiz - Foundation Phase Day {0, 14, 28}`, then `... Day 42+ (rotating)` bi-weekly indefinite | stay useful, no BSS pitch |
| Monthly Market Update | graduates of all 3 tiers + LM2 N3 | `FTHB Monthly Market Update` (1/mo) | long-tail nurture |

## File layout

```
lm1-readiness-filter/
├── _index.md                          (this file)
├── 00-transactional-all-tiers.md      Email 0
├── _shared/
│   └── monthly-market-update.md       Template for graduate nurture
├── tier-a-ready-now/
│   ├── A1-day-00-ready-now-meaning.md
│   ├── A2-day-02-seminole-vs-orange.md
│   ├── A3-day-05-pre-approval-question.md
│   ├── A4-day-09-inspection-contingency.md
│   └── A5-day-14-last-note.md
├── tier-b-ninety-day-sprint/
│   ├── B1-day-00-game-plan.md
│   ├── B2-day-02-before-house-shopping.md
│   ├── B3-day-04-builders-lender.md
│   ├── B4-day-07-where-stuck.md
│   ├── B5-day-12-sprint-graduation.md
│   └── B6-day-21-last-note.md
└── tier-c-foundation/
    ├── C0-day-00-welcome.md
    ├── C1-day-14-credit.md
    ├── C2-day-28-savings.md
    └── C3-plus-rotating-topics.md     (10-topic backlog table)
```

## Load-bearing rules from CLAUDE.md

- **Tier C never gets a BSS pitch**, ever, from any sequence. Hard branch in Pipedrive Workflow Automations.
- **Tier reassignment on retake** — if `fthb_lm1_tier` changes, automation unenrolls the contact from every remaining per-day campaign in the old tier and enrolls them in the new tier starting at Day 0.
- **Tier B who opts into LM2** — when `fthb_received_lm2 = true` flips on a `NINETY_DAY` contact, Pipedrive unenrolls them from every remaining `FTHB Readiness Quiz - 90-Day Sprint Day {N}` campaign and enrolls them in `FTHB 9-Step Roadmap - Delivery (Email 0)` plus the `FTHB 9-Step Roadmap - Nurture Day {3, 7, 14}` series. Prevents the email storm. See `../lm2-process-map/_index.md` for the matching rule.

## Locked language

Never paraphrase, including in email body or subject:

- **"The Orlando First-Time Buyer Readiness Score"**
- Subhead: *"Know in 7 Minutes If You're 30, 90, or 180 Days Away From Your First Home."*

Tier display labels in copy (UI-friendly, not the enum keys): **Ready Now**, **90-Day Sprint**, **Foundation Phase**.
