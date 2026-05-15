# 05 - LM2 The Process Map: Product Spec

This is the build spec for the second lead magnet, the 9-Step First Home Roadmap. It's lighter than LM1 (no scorecard, no scoring math), but the spec still locks copy, routes, payload, and delivery.

---

## Public name (final)

**The 9-Step First Home Roadmap**
*Subhead:* Exactly What Happens Between "I Think I'm Ready" and Keys in Your Hand in Orlando.

Locked phrasing. Don't paraphrase on landing pages or in DMs.

---

## Entry points (where prospects arrive at LM2)

LM2 has exactly two entry routes. **Do not add more.** Fragmenting LM2 into a generic site-wide CTA defeats the segmentation the funnel was built on.

| # | Entry point | Behavior |
|---|---|---|
| 1 | **From LM1 Tier B result page CTA** | Prospect's email is already on file. Opt-in form is **prefilled** with first name + email; they submit with one click. Payload includes a `source: "lm1_tier_b"` flag for analytics. |
| 2 | **Standalone landing page** at `/orlando-homebuying-roadmap` | For prospects who came from a piece of content specifically about *the process* (e.g., an Instagram reel walking through one step). Standard opt-in: name + email + ZIP. |

LM2 is **not** linked from:

- The main agent homepage navigation
- Tier A or Tier C result pages (Tier A goes to BSS, Tier C goes to nurture only)
- The Instagram/Facebook bio (LM1 owns that slot)

---

## High-level user flow

```
[Entry: Tier B result CTA  OR  /orlando-homebuying-roadmap landing]
                 │
                 ▼
[Opt-in form: prefilled (Tier B) or fresh (standalone)]
                 │
                 ▼
[Inline confirmation page: roadmap rendered as a web page]
                 │
                 ▼
[Email follow-up with PDF copy attached]
                 │
                 ▼
[3-email nurture sequence ending in BSS pitch]
```

---

## Routes & screens

| Route | Screen | Notes |
|---|---|---|
| `/orlando-homebuying-roadmap` | Standalone landing page | Entry point #2. Single hero, single CTA. |
| `/orlando-homebuying-roadmap/get` | Opt-in form | Name + email + ZIP + language toggle. Prefilled when source = `lm1_tier_b`. |
| `/orlando-homebuying-roadmap/view` | Inline rendered roadmap | The full 9-step content, rendered as a long-form page. Anchor links per step. |
| `/orlando-homebuying-roadmap/view/es` | Spanish version | Phase 3 deliverable. Same layout, translated content. |

---

## Standalone landing page (`/orlando-homebuying-roadmap`)

### Hero copy

> **The 9-Step First Home Roadmap**
> Exactly What Happens Between "I Think I'm Ready" and Keys in Your Hand in Orlando.
>
> A step-by-step visual map of every milestone, decision, and potential mistake in the Orlando home-buying process, so you walk in knowing more than most buyers who've done this before.
>
> [ Send me the roadmap (free) → ]
>
> *Available in English and Spanish. Delivered in 60 seconds. No phone call required.*

### Sub-hero: "What's in it"

- The 9 Steps from "Check Your Credit" to "Close Day": what *you* do, what *your agent* does, how long each takes
- The 3 places first-time buyers in Orlando lose money (named, specific, avoidable)
- Sanford to Downtown Orlando gotchas: HOA timelines, rainy-season inspection issues, Seminole-vs.-Orange school-zone resale math, the commute trade-off between Sanford and Downtown Orlando
- The Pre-Approval Cheat Sheet: documents to gather + the one credit number that matters more than your score

### Sub-hero: Who it's for

> If you took my Readiness Scorecard and landed in the 90-Day Sprint tier, this is the document I built specifically for you. If you didn't take the scorecard, you can take it later, but the roadmap is useful to anyone in the first 6 months of the home-buying process in Orlando.
>
> [ Or, take the 7-minute Readiness Scorecard first → ]

---

## Opt-in form (`/orlando-homebuying-roadmap/get`)

Fields:

| Field | Required | Notes |
|---|---|---|
| `first_name` | Yes | Prefilled when coming from Tier B |
| `email` | Yes | Prefilled when coming from Tier B |
| `zip` | Yes | Lets the agent confirm the prospect is in the Orlando metro |
| `language` | Yes (radio) | English / Español. Defaults to English. |
| `consent` | Yes (checkbox) | "I'm ok with {{agent}} emailing me the roadmap and occasional follow-ups." |

The form posts to the **same Make.com webhook** as LM1, with a distinguishing `magnet: "lm2"` field. This keeps the Make.com scenario routing logic in one place.

---

## Form payload sent to Make.com

```json
{
  "magnet": "lm2",
  "submitted_at": "2026-05-14T18:45:00Z",
  "contact": {
    "first_name": "Carlos",
    "email": "carlos@example.com",
    "zip": "32750",
    "preferred_language": "es"
  },
  "source": "lm1_tier_b",        // or "standalone"
  "lm1_tier": "NINETY_DAY",      // null if standalone
  "consent": true,
  "utm_source": "instagram_reel_step_2"
}
```

When `source = "lm1_tier_b"`, Make.com is expected to:

1. Skip creating a new contact (one already exists from LM1)
2. Tag the existing contact with `received_lm2 = true`
3. Send the LM2 transactional email immediately
4. Pause the LM1 Tier B nurture sequence (no email storm from both)
5. Enroll the contact in the LM2 nurture sequence starting at email 1

When `source = "standalone"`, Make.com creates a new contact and enrolls them in the LM2 nurture sequence with no LM1 pre-context.

---

## Inline rendered roadmap (`/orlando-homebuying-roadmap/view`)

The roadmap renders as a long-form web page. It is the *same content* as the PDF version (see content draft in `06-process-map-content.md`), but optimized for web reading.

### Page structure

```
[Header bar]
  - "The 9-Step First Home Roadmap" + jump-to-step nav

[Intro paragraph + how to use this document]

[Visual roadmap diagram: 9 numbered steps, vertical for mobile]
  - Each step is a clickable anchor

[Step 1 detail section] (repeats for steps 1–9)
  - What you do
  - What your agent does
  - How long it takes
  - Common mistakes
  - Local Orlando notes

[The 3 Places First-Time Buyers Lose Money]
  - Three named scenarios with specific examples

[Orlando-Specific Gotchas]
  - HOA disclosure timeline
  - Florida rainy-season + roof inspection
  - Seminole vs. Orange school-zone resale math
  - The commute-and-price trade-off between Sanford and Downtown Orlando

[Pre-Approval Cheat Sheet]
  - Documents checklist
  - The "one number that matters more than your score"

[CTA: Buyer Strategy Session]
  - Single CTA: "When you want me to walk this with you..."

[Footer: license, brokerage, bilingual note]
```

### Design specifications

- Anchor links from the visual roadmap diagram into each step section
- Sticky "back to top" / "jump to step" nav on desktop
- Mobile: collapsed accordion per step (tap to expand)
- Print stylesheet that produces a usable hard-copy version (some buyers will print this)
- The visual roadmap is an SVG, not an image; keeps it crisp at any size and lets us update step labels in code, not in Figma

---

## Email-attached PDF version

A PDF of the same content is attached to the delivery email. Two production options; pick one in Phase 2:

**Option A: Static PDF (preferred for Phase 2 MVP)**
The agent designs one English PDF and one Spanish PDF in Canva or similar. Make.com attaches the matching language file. Pro: simple, beautiful. Con: updates require redesign.

**Option B: Generated PDF via Make.com / Cloud Run**
Make.com calls a PDF generation service (e.g., PDFShift, DocRaptor, or a self-hosted Puppeteer instance) that renders the web view as a PDF. Pro: single source of truth. Con: more moving parts, ongoing cost.

**Recommendation:** Ship with Option A. Move to Option B only if the content starts updating monthly.

---

## Analytics events

| Event | When |
|---|---|
| `roadmap_landing_view` | `/orlando-homebuying-roadmap` page load (standalone entry only) |
| `roadmap_optin_view` | `/orlando-homebuying-roadmap/get` page load |
| `roadmap_optin_submit` | Successful form submit (props: source, language) |
| `roadmap_view_view` | `/orlando-homebuying-roadmap/view` page load (props: language) |
| `roadmap_step_expand` | A step section was expanded on mobile (props: step_number) |
| `roadmap_bss_cta_click` | BSS CTA at bottom of roadmap clicked |

---

## Definition of Done: LM2 launch

LM2 ships when all of the following are true:

- [ ] `/orlando-homebuying-roadmap` standalone landing page is live
- [ ] `/orlando-homebuying-roadmap/get` opt-in form posts to Make.com webhook with correct payload (both source types tested)
- [ ] `/orlando-homebuying-roadmap/view` renders all 9 steps + 3 mistakes + gotchas + cheat sheet
- [ ] Tier B result page CTA now links to the prefilled opt-in
- [ ] Make.com handles the "existing LM1 contact" routing correctly (no double-enrollment)
- [ ] Make.com pauses LM1 Tier B sequence when LM2 is received
- [ ] LM2 transactional email delivers within 60 seconds with PDF attached (English version live; Spanish toggle disabled until Phase 3)
- [ ] All 3 LM2 nurture emails are scheduled and rendering correctly
- [ ] Print stylesheet produces a usable printout
- [ ] Mobile accordion works on iOS Safari and Android Chrome
- [ ] Analytics events fire on every step
- [ ] Agent has reviewed the rendered roadmap end-to-end on mobile and desktop
