# LM1 — *The Orlando First-Time Buyer Readiness Score* (email campaigns)

**Source spec:** [docs/04-readiness-filter-emails.md](../../docs/04-readiness-filter-emails.md)

LM1 has **four Pipedrive campaigns** plus the shared monthly market update:

| Campaign | Tier | Emails | Cadence | Goal |
|---|---|---|---|---|
| FTHB LM1 - Transactional (Email 0) | all | 1 | within 60s of submit | deliver result link |
| FTHB LM1 - Tier A | A (READY_NOW) | 5 | Day 0, 2, 5, 9, 14 | book BSS |
| FTHB LM1 - Tier B | B (NINETY_DAY) | 6 | Day 0, 2, 4, 7, 12, 21 | get them into LM2, then BSS |
| FTHB LM1 - Tier C | C (FOUNDATION) | C0, C1, C2, then C3+ rotating | Day 0, then bi-weekly indefinite | stay useful, no BSS pitch |
| FTHB Monthly Market Update | graduates of all 3 tiers + LM2 N3 | 1/mo | monthly | long-tail nurture |

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
- **Tier reassignment on retake** — if `fthb_lm1_tier` changes, automation unenrolls from old-tier campaign and enrolls in new-tier campaign starting at email 1.
- **Tier B who opts into LM2** — when `fthb_received_lm2 = true` flips on a `NINETY_DAY` contact, Pipedrive unenrolls from `FTHB LM1 - Tier B` and enrolls in `FTHB LM2 - Roadmap`. Prevents the email storm. See `../lm2-process-map/_index.md` for the matching rule.

## Locked language

Never paraphrase, including in email body or subject:

- **"The Orlando First-Time Buyer Readiness Score"**
- Subhead: *"Know in 7 Minutes If You're 30, 90, or 180 Days Away From Your First Home."*

Tier display labels in copy (UI-friendly, not the enum keys): **Ready Now**, **90-Day Sprint**, **Foundation Phase**.
