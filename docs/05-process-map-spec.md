
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
| 1 | **From LM1 Tier B result page CTA** | The Tier B result page already has the visitor's first name and email in its URL query params (`?n=…&t=B&s=…`). The CTA is a link to `/orlando-homebuying-roadmap/get?n=Maria&e=maria%40example.com&src=fthb_lm1_tier_b` — the LM2 opt-in form reads those params and pre-fills the fields. No storage; the URL is the carrier. Payload includes `source: "fthb_lm1_tier_b"`. |
| 2 | **Standalone landing page** at `/orlando-homebuying-roadmap` | For prospects who came from a piece of content specifically about *the process* (e.g., an Instagram reel walking through one step). Standard opt-in: name + email + ZIP. |

Pre-filling from URL params is a non-trustworthy hint, not authentication. Make.com still treats the incoming LM2 webhook as the source of truth and looks the contact up in Pipedrive by email.

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
[Opt-in form: prefilled from URL params (Tier B) or fresh (standalone)]
                 │  (on submit: POST to the same Make.com webhook with magnet:"fthb_lm2";
                 │   redirect immediately to the rendered roadmap)
                 ▼
[Inline rendered roadmap (long-form web page) at /orlando-homebuying-roadmap/view]
                 │
                 ▼
[Make.com → Pipedrive Person update → Pipedrive Workflow Automation
 → Pipedrive Campaigns: transactional email with PDF download link]
                 │
                 ▼
[3-email nurture sequence ending in BSS pitch (Pipedrive Campaigns)]
```

---

## Routes & screens

| Route | Screen | Notes |
|---|---|---|
| `/orlando-homebuying-roadmap` | Standalone landing page | Entry point #2. Single hero, single CTA. Static. |
| `/orlando-homebuying-roadmap/get` | Opt-in form | Name + email + ZIP + language toggle. Reads `?n=`, `?e=`, `?src=` from the URL to pre-fill (when coming from the Tier B result page). Vanilla JS handles form submission via `fetch` to the Make.com webhook, then redirects. |
| `/orlando-homebuying-roadmap/view` | Inline rendered roadmap | The full 9-step content, rendered as a long-form page. Anchor links per step. Reachable directly without opting in (the roadmap *is* the lead magnet; the opt-in is what enrolls them in the nurture, not what gates the read). |
| `/assets/orlando-9-step-roadmap.pdf` | Static PDF | Hosted on the Astro site. Linked from the LM2 transactional email. Updates are a redeploy. |

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

---

## Opt-in form (`/orlando-homebuying-roadmap/get`)

Fields:

| Field | Required | Notes |
|---|---|---|
| `first_name` | Yes | Prefilled when coming from Tier B |
| `email` | Yes | Prefilled when coming from Tier B |
| `zip` | Yes | Lets the agent confirm the prospect is in the Orlando metro |
| `consent` | Yes (checkbox) | "I'm ok with {{agent}} emailing me the roadmap and occasional follow-ups." |

The form `fetch`-POSTs to the **same Make.com webhook URL** as LM1, with a distinguishing `magnet: "fthb_lm2"` field. This keeps the Make.com scenario routing logic in one place. The page redirects to `/orlando-homebuying-roadmap/view` immediately without awaiting the response (same fire-and-forget pattern as LM1).

---

## Form payload sent to Make.com

```json
{
  "magnet": "fthb_lm2",
  "submitted_at": "2026-05-14T18:45:00Z",
  "contact": {
    "first_name": "Carlos",
    "email": "carlos@example.com",
    "zip": "32750"
  },
  "source": "fthb_lm1_tier_b",        // or "fthb_lm2_standalone"
  "fthb_lm1_tier": "NINETY_DAY",      // null if standalone
  "consent": true,
  "utm_source": "instagram_reel_step_2"
}
```

When `source = "fthb_lm1_tier_b"`, Make.com is expected to:

1. Write a Google Sheet audit row.
2. Look up the contact in Pipedrive by email (one already exists from LM1; if for some reason it doesn't, fall through to the standalone path).
3. Update the existing Pipedrive Person: set `fthb_received_lm2 = true`, `fthb_lm2_received_at = now`.
4. Return `200`.

A Pipedrive Workflow Automation watches for `fthb_received_lm2` flipping to `true` and:

1. Unenrolls the contact from the LM1 Tier B campaign (the "pause/no-double-storm" rule).
2. Enrolls them in the LM2 transactional + nurture campaign starting at email 0.

When `source = "fthb_lm2_standalone"`, Make.com:

1. Writes the Google Sheet audit row.
2. Looks up the contact in Pipedrive by email.
   - Not found: create a new Person with `fthb_received_lm2 = true`, `fthb_lm2_received_at = now`, and an `fthb_lm2_source = "fthb_lm2_standalone"` tag.
   - Found (rare; usually means a previous LM1 contact who skipped Tier B): update the existing Person with the same flags.
3. A Pipedrive Workflow Automation matched on the standalone path enrolls them in the LM2 transactional + nurture campaign with no LM1 pre-context.

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

[Footer: license, brokerage]
```

### Design specifications

- Anchor links from the visual roadmap diagram into each step section
- Sticky "back to top" / "jump to step" nav on desktop
- Mobile: collapsed accordion per step (tap to expand)
- Print stylesheet that produces a usable hard-copy version (some buyers will print this)
- The visual roadmap is an SVG, not an image; keeps it crisp at any size and lets us update step labels in code, not in Figma

---

## PDF version (download link, not attachment)

The LM2 transactional email links to a **statically hosted PDF** on the Astro site at `/assets/orlando-9-step-roadmap.pdf`.

The agent designs the PDF in Canva (or similar) and drops it into the Astro repo's `/public/assets/` directory. Updates are a redeploy.

Why a download link, not an email attachment:

- Pipedrive Campaigns is the sender, not Make.com. Attaching a dynamic, language-conditional file from Pipedrive Campaigns is awkward; linking to a stable URL is one line of merge-tag-free HTML.
- Static hosting on the Astro site is free (it's part of the existing deploy), the file is CDN-fast for the reader, and the link can be re-shared (the agent will hear "can you send me that PDF again?" — a URL is the answer).
- The transactional email and the inline `/view` page share the same source of truth: whatever's in `/public/assets/` at the most recent deploy.

If PDF *generation* (from the live web view) ever becomes worth it, that would be a Phase 4+ optimization. Not now.

---

## Analytics events

| Event | When |
|---|---|
| `fthb_roadmap_landing_view` | `/orlando-homebuying-roadmap` page load (standalone entry only) |
| `fthb_roadmap_optin_view` | `/orlando-homebuying-roadmap/get` page load |
| `fthb_roadmap_optin_submit` | Successful form submit (props: source, language) |
| `fthb_roadmap_view_view` | `/orlando-homebuying-roadmap/view` page load (props: language) |
| `fthb_roadmap_step_expand` | A step section was expanded on mobile (props: step_number) |
| `fthb_roadmap_bss_cta_click` | BSS CTA at bottom of roadmap clicked |

---

## Definition of Done: LM2 launch

LM2 ships when all of the following are true:

- [ ] `/orlando-homebuying-roadmap` standalone landing page is live
- [ ] `/orlando-homebuying-roadmap/get` opt-in form posts to the Make.com webhook with the correct payload for both source types (`fthb_lm1_tier_b` with URL-prefill, and `fthb_lm2_standalone`)
- [ ] `/orlando-homebuying-roadmap/view` renders all 9 steps + 3 mistakes + gotchas + cheat sheet
- [ ] Tier B result page CTA links to `/orlando-homebuying-roadmap/get` with `?n=&e=&src=fthb_lm1_tier_b` URL params
- [ ] `/assets/orlando-9-step-roadmap.pdf` is in the repo and reachable at the public URL after deploy
- [ ] Make.com correctly handles the "existing LM1 contact" path (Pipedrive lookup by email, update not create)
- [ ] Pipedrive Workflow Automation unenrolls the contact from LM1 Tier B and enrolls them in LM2 when `fthb_received_lm2` flips to `true`
- [ ] LM2 transactional email delivers within 60 seconds of submit, with a working PDF download link
- [ ] All 3 LM2 nurture emails are scheduled in Pipedrive Campaigns and rendering correctly
- [ ] Print stylesheet produces a usable printout of `/view`
- [ ] Mobile accordion works on iOS Safari and Android Chrome
- [ ] Analytics events fire on every step (client-side, not via Make.com)
- [ ] Agent has reviewed the rendered roadmap end-to-end on mobile and desktop
