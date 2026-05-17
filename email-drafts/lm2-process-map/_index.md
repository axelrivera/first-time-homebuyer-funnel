# LM2 — *The 9-Step First Home Roadmap* (email campaign)

**Source spec:** [docs/07-process-map-emails.md](../../docs/07-process-map-emails.md)

LM2 is a **single Pipedrive campaign** — `FTHB LM2 - Roadmap` — used for both `source` values (`fthb_lm1_tier_b` and `fthb_lm2_standalone`) because the roadmap content level-sets all readers.

| Email | Day | Subject | Status target |
|---|---|---|---|
| 0 (transactional) | 0 | Your 9-Step Orlando Home Roadmap (link + PDF inside) | POLISHED before launch |
| N1 | 3 | Which step did you start on? | POLISHED before launch |
| N2 | 7 | The Step 3 → Step 4 jump | POLISHED before launch |
| N3 | 14 | When the roadmap turns into a conversation | POLISHED before launch |

## File layout

```
lm2-process-map/
├── _index.md            (this file)
├── 00-transactional.md  Email 0
└── nurture/
    ├── N1-day-03-which-step.md
    ├── N2-day-07-step-3-to-4.md
    └── N3-day-14-roadmap-to-conversation.md
```

## Tier B email-storm rule (load-bearing)

Anyone arriving at LM2 via `source = fthb_lm1_tier_b` is **already enrolled** in `FTHB LM1 - Tier B`. Without careful routing they will get two parallel sequences.

The rule, split across Make.com and Pipedrive Workflow Automations:

**Make.com (on `magnet: "fthb_lm2"` + `source: "fthb_lm1_tier_b"`):**

1. Webhook receives the payload.
2. Write the Google Sheet audit row.
3. Look up the Pipedrive Person by email.
4. Update the Person: `fthb_received_lm2 = true`, `fthb_lm2_received_at = now`, language toggle if changed.
5. Return `200`.

**Pipedrive Workflow Automation (on `fthb_received_lm2` flipped to `true`):**

1. Unenroll the contact from `FTHB LM1 - Tier B` (cancels all scheduled future emails).
2. Enroll the contact in `FTHB LM2 - Roadmap` starting at Email 0.
3. Campaign scheduling drips N1, N2, N3 on Day 3, 7, 14.

A contact who never opts into LM2 stays in `FTHB LM1 - Tier B` and finishes it normally. No automation fires.

## Post-sequence routing

After N3, the contact moves to the **monthly market-update list** (same destination as Tier A after A5 and Tier B after B6). Template at `../lm1-readiness-filter/_shared/monthly-market-update.md`.

## Locked language

Never paraphrase:

- **"The 9-Step First Home Roadmap"**
- Subhead: *"Exactly What Happens Between 'I Think I'm Ready' and Keys in Your Hand in Orlando."*
