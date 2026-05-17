
The wiring sweep for LM2: every LM2 event is firing exactly once, at the right moment, with the right props.

## Prereqs

- [02-04 Static PDF asset](./04-static-pdf-asset.md) (everything else LM2 needs is in place).
- [00-01 Audit existing site](../00-foundations/01-audit-existing-site.md) (analytics helper signature).

## Goal

All 6 LM2 events fire correctly. No duplicates. Props match below. Combined with the LM1 events from [01-10](../01-lm1-readiness-filter/10-analytics-events.md), the full funnel is measurable.

## Event checklist (LOCKED — do not add or remove)

| Event | When | Props | Where (file) |
|---|---|---|---|
| `fthb_roadmap_landing_view` | `/orlando-homebuying-roadmap` page load (standalone entry only) | none | `src/pages/orlando-homebuying-roadmap/index.astro` |
| `fthb_roadmap_optin_view` | `/orlando-homebuying-roadmap/get` page load | `{ source: 'fthb_lm1_tier_b' \| 'fthb_lm2_standalone' }` | `src/pages/orlando-homebuying-roadmap/get.astro` |
| `fthb_roadmap_optin_submit` | Form submit succeeds (after client-side validation) | `{ source: 'fthb_lm1_tier_b' \| 'fthb_lm2_standalone' }` | `src/scripts/lm2-optin.ts` |
| `fthb_roadmap_view_view` | `/orlando-homebuying-roadmap/view` page load | none | `src/pages/orlando-homebuying-roadmap/view.astro` |
| `fthb_roadmap_step_expand` | A step `<details>` element opens | `{ step_number: 1 \| 2 \| ... \| 9 }` | `src/pages/orlando-homebuying-roadmap/view.astro` (toggle listener) |
| `fthb_roadmap_bss_cta_click` | BSS CTA at the bottom of `/view` is clicked | none | `src/pages/orlando-homebuying-roadmap/view.astro` |

## Implementation notes

- **`fthb_roadmap_optin_view` props.** The `source` prop is the most useful split for analyzing conversion (Tier B-driven opt-ins behave differently than standalone). Add it — it costs nothing. Source resolution mirrors the form's: `?src=fthb_lm1_tier_b` → `'fthb_lm1_tier_b'`; anything else → `'fthb_lm2_standalone'`.
- **`fthb_roadmap_step_expand` fires every time.** If the user opens, closes, then re-opens step 3, that's two events. If dedupe becomes important, do it in the analytics layer; the client just fires raw.
- **The view page might be entered via anchor link** (`/view#step-4`). The auto-scroll behavior is browser-native; don't fire `fthb_roadmap_step_expand` from the URL hash. The `toggle` event on `<details>` is the only signal.
- **No event from the LM2 transactional email.** Email-click tracking lives in Pipedrive Campaigns, not in the Astro analytics stack.
- **No PII in props.** Same rule as LM1 — emails, ZIPs, names belong in Pipedrive.
- **No relay through Make.com.** Direct-from-browser only.

## Things NOT to do

- Don't add new events beyond the table above. Same rule as LM1.
- Don't include emails, ZIPs, or names in event props.
- Don't fire analytics from the Make.com webhook payload path.

## Definition of Done

- [ ] Each event in the table maps to exactly one call site
- [ ] An end-to-end LM2 flow (landing → opt-in → view → BSS click) produces this sequence:
  1. `fthb_roadmap_landing_view` (only if entering from the standalone landing)
  2. `fthb_roadmap_optin_view` (with `source`)
  3. `fthb_roadmap_optin_submit` (with `source`)
  4. `fthb_roadmap_view_view`
  5. `fthb_roadmap_step_expand` × however many sections the user opens
  6. `fthb_roadmap_bss_cta_click` (if the bottom CTA is clicked)
- [ ] A Tier B entry (`/get?n=...&src=fthb_lm1_tier_b`, skipping the standalone landing) does **not** fire `fthb_roadmap_landing_view` — only `fthb_roadmap_optin_view` and onward
- [ ] No `roadmap_*` events fire on `?preview=`-style URLs (if any are added later)

## Verification

In the analytics Realtime view:
- Land on `/orlando-homebuying-roadmap` → see `fthb_roadmap_landing_view`
- Click through to `/get` → see `fthb_roadmap_optin_view`
- Submit → see `fthb_roadmap_optin_submit` and (after redirect) `fthb_roadmap_view_view`
- Open three step accordions → see three `fthb_roadmap_step_expand` events
- Click the bottom BSS CTA → see `fthb_roadmap_bss_cta_click`

Separately, simulate the Tier B path: load `/get?n=Maria&src=fthb_lm1_tier_b` directly. Confirm no `fthb_roadmap_landing_view` fires (only `fthb_roadmap_optin_view`).
