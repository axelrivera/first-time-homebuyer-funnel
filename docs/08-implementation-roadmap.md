Phased build plan for shipping the two-step funnel. Designed for one builder (the agent, who is also an iOS developer with Ruby/Rails backend chops) working in evening/weekend windows.

The principle: **ship Phase 1 fast and start collecting real submissions. Iterate on copy and tier logic against actual responses, not theories.**

---

## Phase summary

| Phase                        | What ships                                                                                                     | Calendar target | Why this phase, this scope                                                                      |
| ---------------------------- | -------------------------------------------------------------------------------------------------------------- | --------------- | ----------------------------------------------------------------------------------------------- |
| **0. Foundations**           | Domain, hosting, Astro + Tailwind scaffold, Make.com webhook, Pipedrive + Campaigns addon, calendar, analytics | Week 1          | The stack is locked (Pipedrive + Make.com + Astro + Tailwind); Phase 0 is wiring, not deciding. |
| **1. LM1 MVP**               | Readiness Filter landing → quiz → result page → transactional email via Pipedrive                              | Weeks 2–4       | This is the whole top of the funnel. Nothing else matters until this works.                     |
| **2. LM1 Nurture + LM2 MVP** | Tier-specific Pipedrive Campaigns live; LM2 standalone landing + delivery                                      | Weeks 5–7       | Now we can capture and nurture across the full segmentation                                     |
| **3. Polish**                | Monthly market-update list, analytics dashboards                                                               | Weeks 8–9       | Operational rhythm + measurement                                                                |
| **4. Iterate**               | Continuous A/B testing of question wording, tier thresholds, CTAs                                              | Ongoing         | Real data > assumptions                                                                         |

Total time to a fully-shipped funnel: roughly **9 weeks** at evening/weekend pace.

---

## Phase 0. Foundations (Week 1)

### Outcomes

By end of week:

- Domain registered, DNS pointed
- Astro + Tailwind project scaffolded, deployed to a static host
- Make.com account active with one test scenario receiving a webhook and writing a Google Sheet row
- Pipedrive account active with the Campaigns addon installed and custom fields in place
- Calendar booking tool live for the BSS
- Analytics provider script in the base layout

### The stack is locked. Phase 0 is wiring, not picking.

| Component          | Choice                                                          | Notes                                                                                                                                         |
| ------------------ | --------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------- |
| Frontend framework | **Astro** + **Tailwind CSS**                                    | Vanilla JS in islands where interactivity is needed (quiz, opt-in forms). No React/Vue/Svelte.                                                |
| Hosting            | **Vercel** (recommended; Netlify or Cloudflare Pages also fine) | Pick whichever has the cleanest preview-deploy story for the agent. Static-only site, so any of them work.                                    |
| Form handler       | **Make.com** webhook (one webhook URL, baked into the build)    | LM1 + LM2 both POST here, distinguished by `magnet`. No env vars; the URL is in the bundle.                                                   |
| CRM                | **Pipedrive**                                                   | Person records hold contact info + custom fields (`fthb_lm1_tier`, `fthb_lm1_display_score`, `fthb_received_lm1`, `fthb_received_lm2`, etc.). |
| Email sending      | **Pipedrive Campaigns** addon                                   | All transactional + nurture lives in named Pipedrive Campaigns. Workflow Automations enroll/unenroll based on field changes.                  |
| Routing logic      | **Pipedrive Workflow Automations**                              | Tier → campaign mapping, LM2-pauses-Tier-B, retake handling. **Not** in Make.com.                                                             |
| PDF for LM2        | Static file in `public/assets/`                                 | Linked from the LM2 transactional email. Updates are a redeploy.                                                                              |
| Calendar           | **Cal.com** (open source, self-hostable)                        | Calendly works equally well; just a URL.                                                                                                      |
| Analytics          | **Plausible** or **PostHog**                                    | Plausible if you want minimal; PostHog if you want funnel analysis from day one. Script tag in `BaseLayout.astro`.                            |

### Deliverables checklist

- [ ] `orlandohomes.example.com` (or whatever the agent's brand is) resolves to a live "Coming soon: Readiness scorecard launching {{date}}" page
- [ ] Astro + Tailwind repo created on GitHub, deployed to host, auto-deploy on `main`
- [ ] Make.com scenario exists with a webhook URL; logs incoming payload, writes a row to a Google Sheet, returns `200`
- [ ] Pipedrive account has the Campaigns addon active and sender authentication (SPF + DKIM) set on the agent's domain
- [ ] Pipedrive custom fields created on the Person object: `fthb_lm1_tier` (enum), `fthb_lm1_display_score` (number), `fthb_lm1_retaken_at` (date), `fthb_received_lm1` (bool), `fthb_received_lm2` (bool), `fthb_lm2_received_at` (date), `fthb_lm2_source` (enum: `fthb_lm1_tier_b` / `fthb_lm2_standalone`), one per-answer field (`fthb_q1_credit_range` ... `fthb_q10_lender`) as enums. Make.com maps the unprefixed JSON keys under `payload.answers.*` to the prefixed Pipedrive field names; the JSON payload itself doesn't carry the prefix because the keys are already scoped by the `magnet` discriminator.
- [ ] Calendar tool has a 30-minute BSS slot type configured; link is in hand for use as `{{book_bss_link}}`
- [ ] Analytics script is in the Astro base layout

---

## Phase 1. LM1 MVP (Weeks 2–4)

### Scope

Ship the Readiness Filter end-to-end: landing page, 10-question quiz, email gate, result page (all 3 tiers), one transactional email per tier. **No nurture sequences yet**. Those come in Phase 2. The goal is to be able to send a friend the URL and have them get a real, signed result page.

### Week 2. Quiz front-end

- [ ] Build `/orlando-homebuying-readiness-quiz` landing page from `02-readiness-filter-spec.md` copy (static Astro page, Tailwind classes)
- [ ] Build `/orlando-homebuying-readiness-quiz/start` as **one** Astro page with a vanilla-JS state machine that holds all 10 questions + the email gate; question switching is DOM show/hide, not page navigation; state is in-memory only (no `localStorage`, no cookies, no `sessionStorage` for in-flight answers — `sessionStorage` is written only after a successful submit, and only to carry `first_name` + `email` to the LM2 Roadmap opt-in form)
- [ ] Create `src/config/readiness.ts` (or `.json`) and put all tweakable parameters there (thresholds, point values, override actions). See "Configuration" section in `02-readiness-filter-spec.md` for the full list.
- [ ] Implement the scoring engine as a TypeScript pure function (no UI deps) that reads config rather than hardcoded values
- [ ] Walk the 6 canonical scoring examples (below) through the engine by hand and confirm each lands in the expected tier

### Week 3. Result page + Make.com → Pipedrive integration

- [ ] On submit: compute tier/score client-side, fire-and-forget `fetch` POST to Make.com, redirect immediately to `/orlando-homebuying-readiness-quiz/result?n=…&t=A|B|C&s=…`
- [ ] Build `/orlando-homebuying-readiness-quiz/result` page that reads query params and renders the matching tier (A, B, or C). Malformed params → "your result has been emailed to you" fallback view. Support `?preview=A|B|C` for agent QA.
- [ ] Set up Make.com scenario:
  - Webhook trigger receives the form payload
  - Writes one row to a Google Sheet (cheap, queryable backup; also the failure-recovery audit)
  - Looks up the Person in Pipedrive by email; creates or updates with `fthb_lm1_tier`, `fthb_lm1_display_score`, `fthb_received_lm1`, the answer fields, and `fthb_lm1_retaken_at` if applicable
  - Returns `200`
- [ ] Build the three transactional Pipedrive Campaigns (one per tier) from `04-readiness-filter-emails.md` (Email 0 only this week; the nurture chain comes in Phase 2)
- [ ] Build the Pipedrive Workflow Automation: `fthb_received_lm1 = true` → branch on `fthb_lm1_tier` → enroll in the matching transactional campaign

### Week 4. Polish and ship

- [ ] Build the Orlando market snapshot component, pulling from a local JSON file in the repo
- [ ] First populate of the market snapshot data (manual MLS pull)
- [ ] Mobile QA across iOS Safari, Android Chrome, desktop Safari, Chrome, Firefox
- [ ] Accessibility QA (keyboard nav, screen reader, color contrast)
- [ ] Agent takes the quiz themselves end-to-end and approves voice/copy
- [ ] Verify all 4 tier overrides fire correctly on real submissions
- [ ] Set up scheduled task: monthly market snapshot update reminder
- [ ] **LAUNCH:** Update the `/orlando-homebuying-readiness-quiz` URL in the agent's Instagram bio, Facebook bio, email signature

### Canonical scoring examples

| #   | Scenario                                                                                 | Expected tier | Expected behavior                               |
| --- | ---------------------------------------------------------------------------------------- | ------------- | ----------------------------------------------- |
| 1   | All-high answers (740+, $40K saved, 2+ years W-2, 30-day timeline, pre-approved)         | Tier A        | Display score = 100 (raw 89/89)                 |
| 2   | Mid-range across the board (680-739, $10–20K, 1-2 years W-2, 3-6 months, lender in mind) | Tier B        | Display score in the 60–75 range                |
| 3   | Low everything, 1+ year timeline, no lender contact                                      | Tier C        | Display score below 30                          |
| 4   | High score except Q1 = `unknown`                                                         | Tier B        | `credit_unknown_or_low` override demotes from A |
| 5   | High score except Q9 = `exploring`                                                       | Tier C        | Timeline override caps at C                     |
| 6   | High score except Q7 = `between_jobs`                                                    | Tier C        | Employment override caps at C                   |

### Definition of Done for Phase 1

The full LM1 launch DoD lives in `02-readiness-filter-spec.md`. Re-checked here for prominence:

- All 6 canonical scoring examples produce the expected tier and score
- A friend can take the quiz on their phone and receive the right transactional email from Pipedrive Campaigns within 60 seconds
- The agent has personally tested submitting all 3 tier-result scenarios end-to-end (Astro → Make.com → Pipedrive → Pipedrive Campaigns)

---

## Phase 2. LM1 Nurture + LM2 MVP (Weeks 5–7)

### Scope

Two parallel workstreams: build out the LM1 email nurture sequences for all 3 tiers, AND ship the LM2 Process Map.

### Week 5. LM1 nurture campaigns

- [ ] Set up `FTHB LM1 - Tier A` campaign in Pipedrive Campaigns (5 emails from `04-readiness-filter-emails.md`)
- [ ] Set up `FTHB LM1 - Tier B` campaign (6 emails)
- [ ] Set up `FTHB LM1 - Tier C` campaign (welcome + bi-weekly rotation of 10 topics)
- [ ] Build the Pipedrive Workflow Automation: on `fthb_received_lm1 = true`, branch on `fthb_lm1_tier` and enroll in the matching campaign at email 1; on `fthb_lm1_tier` change (retake), unenroll from the old tier campaign and enroll in the new one
- [ ] Test enrollment: submit one fake response per tier, confirm the right campaign fires
- [ ] Set the BSS calendar link on Tier A emails to use UTM tracking so we can measure conversion

### Week 6. LM2 build

- [ ] Build `/orlando-homebuying-roadmap` standalone landing page from `05-process-map-spec.md`
- [ ] Build `/orlando-homebuying-roadmap/get` opt-in form (read `?n=`, `?e=`, `?src=` URL params for pre-fill)
- [ ] Build `/orlando-homebuying-roadmap/view` rendered roadmap from `06-process-map-content.md`
- [ ] Design and produce the static English PDF version (Canva); drop it at `public/assets/orlando-9-step-roadmap.pdf`
- [ ] Wire Make.com to handle `magnet: "fthb_lm2"` payloads: look up the Pipedrive Person, set `fthb_received_lm2`, `fthb_lm2_received_at`, `fthb_lm2_source`
- [ ] Build the Pipedrive Workflow Automation: on `fthb_received_lm2 = true` for a contact with `fthb_lm1_tier = NINETY_DAY`, unenroll from `FTHB LM1 - Tier B` and enroll in `FTHB LM2 - Roadmap`
- [ ] Build the `FTHB LM2 - Roadmap` transactional email (Email 0 in `07-process-map-emails.md`) — links to the rendered roadmap and to the static PDF; no email attachment

### Week 7. LM2 nurture + end-to-end checks

- [ ] Add the 3 nurture emails (N1, N2, N3) to the `FTHB LM2 - Roadmap` campaign on the cadence in `07-process-map-emails.md`
- [ ] Update the Tier B result page CTA to link to `/orlando-homebuying-roadmap/get?n=…&e=…&src=fthb_lm1_tier_b`
- [ ] Update Tier A result CTA copy to mention "if you want the roadmap too, it's here"
- [ ] End-to-end smoke check:
  - Submit LM1 as Tier B → receive Tier B Email 0 → click CTA → opt into LM2 → receive LM2 transactional → confirm in Pipedrive that the contact is no longer scheduled for any remaining LM1 Tier B emails
  - Submit LM2 as standalone → receive LM2 transactional → confirm new Pipedrive Person is tagged correctly

### Definition of Done for Phase 2

- All 3 LM1 Pipedrive Campaigns send on schedule for fake submissions in each tier
- LM2 opt-in works from both entry points and the right Pipedrive automation fires for each
- A Tier B contact does not receive duplicate emails when they opt into LM2 (the LM1 Tier B campaign cleanly unenrolls them)
- All Pipedrive Campaigns merge fields resolve correctly (no `*|FIRST_NAME|*` showing up as literal text)
- Static PDF is downloadable from the link in the LM2 transactional
- Agent has reviewed every email in every campaign via Pipedrive Campaigns' preview-send to their own inbox

---

## Phase 3. Polish (Weeks 8–9)

### Scope

Operational rhythm and measurement. The two pieces that make the funnel keep working without the agent having to think about it: the monthly nurture list (so graduates from every sequence stay warm) and the funnel-metrics dashboard (so Phase 4 iteration has signal to work from).

### Week 8. Monthly market-update list

- [ ] Set up the long-term `FTHB Monthly Market Update` campaign in Pipedrive Campaigns
- [ ] Build a Pipedrive Workflow Automation: when a contact finishes any of the per-tier or LM2 campaigns, enroll them in `FTHB Monthly Market Update`
- [ ] Draft the first 3 monthly emails (so the agent isn't writing them under deadline)
- [ ] Set a scheduled task reminder for the 1st of each month to write the next one

### Week 9. Analytics + iteration setup

- [ ] Build a simple dashboard (or Google Sheet) tracking:
  - Daily LM1 starts vs. completes
  - Tier distribution by week
  - LM2 opt-in rate from Tier B
  - BSS booking rate from Tier A
  - Email open + click rates per sequence
- [ ] Identify the lowest-converting step (likely the email gate) and queue an A/B test
- [ ] Set up a scheduled task: weekly summary of funnel metrics to the agent's email

### Definition of Done for Phase 3

- The agent receives a weekly funnel-metrics email
- The monthly market-update list has 3 emails pre-drafted and the first one scheduled to send
- Every graduating contact from LM1 Tier A/B/C and LM2 nurture lands in the monthly list automatically

---

## Phase 4. Iterate (Ongoing)

### Cadence

- **Weekly:** Skim funnel metrics. Identify the worst-performing step. Run one test.
- **Monthly:** Update the Orlando market snapshot table. Write the next monthly nurture email. Review tier distribution. Are the thresholds catching the right segments?
- **Quarterly:** Look at BSS booking outcomes. Did the prospect actually buy? Did they buy with you? Trace successful buyers back through the funnel. What did they do differently?

### Things worth A/B testing in order of likely impact

1. **The 7-minute promise**. Does "5 minutes" convert better? Or worse, because it's less believable?
2. **The email-gate placement**. What happens if we put it on Q5 instead of Q11? (Hypothesis: way worse, but should be measured.)
3. **The Tier B CTA copy**. "Send me the 9-Step Roadmap" vs. "Get the 90-Day Game Plan"
4. **The Tier A CTA copy**. "Book the Buyer Strategy Session" vs. "Show me what's available in my range right now"
5. **Question order**. Does putting Q9 (timeline) first surface tier earlier and improve completion?

Each test runs for a minimum of 50 completed submissions per arm before calling a winner. Below that volume, the noise is too large to trust the signal.

### What NOT to do during iteration

- Don't add new lead magnets. The funnel works because there are exactly two. Adding a third splits attention and almost always lowers conversion of the existing two.
- Don't redesign before measuring. The first instinct after launch is to rebuild things. Measure first.
- Don't paid-advertise to compensate for low organic volume. The Hormozi rules in this funnel are non-negotiable. Slow organic growth is the trade-off and the long-term moat.

---

## Risks and mitigations

| Risk                                                                 | Mitigation                                                                                                                                                                                                                               |
| -------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Make.com hits its operations limit on a free plan                    | Upgrade to the Pro plan when volume approaches the free-tier ceiling. Forecast around 5–10 ops per submission (webhook → sheet → Pipedrive lookup → Pipedrive update).                                                                   |
| Pipedrive Campaigns flags the agent's domain for low engagement      | Warm up the sending domain by sending the first 50 emails to a list of friends/family who will open. After that, organic volume protects reputation. Make sure SPF/DKIM/DMARC are set on the agent's domain.                             |
| Agent runs out of evening hours to ship                              | Phases 1, 2, 3 can each slip by 2 weeks without breaking the funnel. Don't shortcut quality on the questions, scoring logic, or result-page copy; those are the asset.                                                                   |
| Make.com webhook fails silently and submissions get lost             | Make.com writes a Google Sheet audit row _before_ the Pipedrive call. If Pipedrive errors, the row is still there and the agent can manually recover the contact within 24 hours.                                                        |
| Make.com webhook URL gets scraped from the static bundle and spammed | Add a shared-token field to the payload (also baked into the build) that Make.com verifies before processing. Rotate by redeploying. **Do not** introduce env vars or a backend to "hide" the URL — the threat model doesn't justify it. |
| Result page query params are tampered with                           | Acceptable. The page renders from self-reported data; the authoritative record is in Pipedrive (from the webhook). No state on the static site keys off the result-page URL.                                                             |

---

## What we're explicitly not building

These came up during planning and were deferred. Documenting so they don't get re-litigated mid-build.

- A self-hosted CRM (Rails or otherwise). Pipedrive is enough for Year 1.
- A backend, even a "tiny" one. The point of Make.com + Pipedrive is to avoid this; if a requirement seems to need a backend, the requirement is wrong.
- Cookies, `localStorage`, or environment variables on the static site. These bring compliance and operational complexity that the funnel does not need. (`sessionStorage` is used only for the narrow LM1 → LM2 prefill bridge: `first_name` + `email` written on quiz submit, read by the Roadmap opt-in form, cleared when the tab closes.)
- A custom-built calendar tool. Cal.com or Calendly handles BSS booking. Building one is a distraction.
- A buyer's portal where prospects can log in and "see their progress." This sounds great and adds nothing to conversion in Year 1.
- Native iOS / Android apps. The funnel is web-only. The agent's iOS skills are _not_ in scope for this project.
- Anything paid (ads, lead lists, etc.). Hard rule from the project knowledge base.
