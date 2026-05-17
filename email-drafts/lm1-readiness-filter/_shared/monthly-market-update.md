---
id: SHARED-MONTHLY
funnel: fthb
magnet: shared
tier: graduates (Tier A after A5, Tier B after B6, Tier C, LM2 after N3)
campaign: FTHB Monthly Market Update
cadence: 1x / month
goal: Stay useful, stay top-of-mind, soft CTAs
status: TEMPLATE       # DRAFT | REVIEWING | POLISHED | SHIPPED | TEMPLATE
last_edited: 2026-05-17
source_doc: docs/04-readiness-filter-emails.md (and docs/07)
---

# Monthly Market Update — template

Every contact eventually lands here, regardless of which sequence they came from.

## Required structure for each monthly send

1. **One short market data update** — median price for one Seminole or Orange County area along the I-4 corridor (Sanford → Downtown Orlando), with what the move means for FTHBs in that area.
2. **One useful tip** — financing, inspections, neighborhood research, etc. (rotate from the Tier C topic list in `tier-c-foundation/C3-plus-rotating-topics.md`).
3. **One soft CTA** — *never* the BSS by default. Examples:
   - "Want to retake the scorecard? Your numbers may have moved." → `{{fthb_retake_link}}`
   - "Want the roadmap if you haven't seen it yet?" → `{{fthb_lm2_optin_link}}`
   - "Replying to this email is the fastest way to get a question answered."

## Quarterly BSS pitch rule

The BSS gets pitched in the monthly email **once per quarter** (March, June, September, December), and only as a P.S.:

> *P.S. If you want to talk this quarter, here's the link:* `{{book_bss_link}}`

## Tier C exception (load-bearing)

Tier C contacts are **never** pitched the BSS from any sequence — including the monthly update — until they cross into Tier B/A by retaking the scorecard. The quarterly P.S. above is suppressed for any contact whose current `fthb_lm1_tier = FOUNDATION`. Implement as a hard branch in the Pipedrive Workflow Automation, not as something the copy has to remember.

## Polish notes

<!-- Use this section to capture your edits, alternates, and decisions. -->

- [ ] Decide first 6 months of monthly topic rotation and link to drafts here
- [ ] Lock the on-corridor area rotation: Sanford, Lake Mary, Longwood, Altamonte, Maitland, Winter Park, Downtown, Oviedo, Winter Springs, Casselberry, Apopka, Eatonville, College Park
