# 08. Implementation Roadmap

Phased build plan for shipping the two-step funnel. Designed for one builder (the agent, who is also an iOS developer with Ruby/Rails backend chops) working in evening/weekend windows.

The principle: **ship Phase 1 fast and start collecting real submissions. Iterate on copy and tier logic against actual responses, not theories.**

---

## Phase summary

| Phase | What ships | Calendar target | Why this phase, this scope |
|---|---|---|---|
| **0. Foundations** | Domain, hosting, Make.com account, email sending tool chosen | Week 1 | One week to remove every tool decision from the critical path |
| **1. LM1 MVP** | Readiness Filter landing → quiz → result page → transactional email | Weeks 2–4 | This is the whole top of the funnel. Nothing else matters until this works. |
| **2. LM1 Nurture + LM2 MVP** | Tier-specific email sequences live; LM2 standalone landing + delivery | Weeks 5–7 | Now we can capture and nurture across the full segmentation |
| **3. Spanish + Polish** | Spanish content for LM1 & LM2, monthly market-update list, analytics dashboards | Weeks 8–10 | Bilingual edge + operational rhythm |
| **4. Iterate** | Continuous A/B testing of question wording, tier thresholds, CTAs | Ongoing | Real data > assumptions |

Total time to a fully-shipped funnel: roughly **10 weeks** at evening/weekend pace.

---

## Phase 0. Foundations (Week 1)

### Outcomes

By end of week:

- Domain registered, DNS pointed
- Astro project scaffolded, deployed to a host (Vercel, Netlify, or Cloudflare Pages; pick fastest to start)
- Make.com account active with one test scenario receiving a webhook
- Email sending tool selected and verified
- Calendar booking tool selected for the BSS
- Analytics provider decided

### Decisions to make this week (don't defer)

| Decision | Recommended default | Reason to pick something else |
|---|---|---|
| Hosting | Vercel | If you want zero-config preview deployments per branch |
| Email sender | **Mailchimp** or **ConvertKit** (Kit) | Pick ConvertKit if you expect to lean hard on tag-based segmentation; Mailchimp if you want the simplest WYSIWYG |
| PDF generation | Static PDFs designed in Canva | Move to generated PDFs only if content updates more than monthly |
| Calendar | **Cal.com** (open source, self-hostable) | Calendly if you want zero setup |
| Analytics | **Plausible** or **PostHog** | Plausible for simplicity; PostHog if you want event-level funnel analysis from day one |

### Deliverables checklist

- [ ] `orlandohomes.example.com` (or whatever the agent's brand is) resolves to a live "Coming soon: Readiness scorecard launching {{date}}" page
- [ ] Astro repo created on GitHub, deployed to host, auto-deploy on `main`
- [ ] Make.com scenario exists with a webhook URL, logs incoming payload, returns `200`
- [ ] Email sender account exists with sender authentication (SPF + DKIM) set on the domain
- [ ] Calendar tool has a 30-minute BSS slot type configured
- [ ] Analytics script is in the Astro base layout

---

## Phase 1. LM1 MVP (Weeks 2–4)

### Scope

Ship the Readiness Filter end-to-end: landing page, 10-question quiz, email gate, result page (all 3 tiers), one transactional email per tier. **No nurture sequences yet**. Those come in Phase 2. The goal is to be able to send a friend the URL and have them get a real, signed result page.

### Week 2. Quiz front-end

- [ ] Build `/orlando-homebuying-readiness-quiz` landing page from `02-readiness-filter-spec.md` copy
- [ ] Build `/orlando-homebuying-readiness-quiz/start` Astro island (the 10-question quiz)
- [ ] Implement the form state machine: 10 questions + email gate + submit
- [ ] Create `src/config/readiness.ts` (or `.json`) and put all tweakable parameters there (thresholds, point values, override actions, token expiry). See "Configuration" section in `02-readiness-filter-spec.md` for the full list.
- [ ] Implement the scoring engine as a TypeScript pure function (testable, no UI deps) that reads config rather than hardcoded values
- [ ] Write unit tests for scoring: at least 1 case per tier + 1 case per override (6 canonical cases below)
- [ ] Build `/orlando-homebuying-readiness-quiz/contact` email gate

### Week 3. Result + Make.com integration

- [ ] Implement signed-token generation on form submit (HMAC-SHA256, 30-day expiry)
- [ ] Build `/orlando-homebuying-readiness-quiz/result` page that decodes token and renders all 3 tiers
- [ ] Build `/orlando-homebuying-readiness-quiz/result/preview` for agent QA
- [ ] Set up Make.com scenario:
  - Webhook trigger receives the form payload
  - Logs to a Google Sheet (cheap, queryable backup)
  - Sends Tier A / B / C transactional email
  - Tags contact in email tool with `tier_*`, `received_lm1 = true`
- [ ] Wire all 3 tier transactional emails from `04-readiness-filter-emails.md`

### Week 4. Polish and ship

- [ ] Build the Orlando market snapshot component, pulling from a local JSON file in the repo
- [ ] First populate of the market snapshot data (manual MLS pull)
- [ ] Mobile QA across iOS Safari, Android Chrome, desktop Safari, Chrome, Firefox
- [ ] Accessibility QA (keyboard nav, screen reader, color contrast)
- [ ] Agent takes the quiz themselves end-to-end and approves voice/copy
- [ ] Test all 4 tier overrides with real submissions
- [ ] Set up scheduled task: monthly market snapshot update reminder
- [ ] **LAUNCH:** Update the `/orlando-homebuying-readiness-quiz` URL in the agent's Instagram bio, Facebook bio, email signature

### Canonical test cases for scoring

| # | Scenario | Expected tier | Expected behavior |
|---|---|---|---|
| 1 | All-high answers (740+, $40K saved, 2+ years W-2, 30-day timeline, pre-approved) | Tier A | Display score = 100 (raw 89/89) |
| 2 | Mid-range across the board (680-739, $10–20K, 1-2 years W-2, 3-6 months, lender in mind) | Tier B | Display score in the 60–75 range |
| 3 | Low everything, 1+ year timeline, no lender contact | Tier C | Display score below 30 |
| 4 | High score except Q1 = `unknown` | Tier B | `credit_unknown_or_low` override demotes from A |
| 5 | High score except Q9 = `exploring` | Tier C | Timeline override caps at C |
| 6 | High score except Q7 = `between_jobs` | Tier C | Employment override caps at C |

### Definition of Done for Phase 1

The full LM1 launch DoD lives in `02-readiness-filter-spec.md`. Re-checked here for prominence:

- All 6 canonical test cases produce the expected tier and score
- A friend can take the quiz on their phone and receive the right transactional email within 60 seconds
- The agent has personally tested submitting all 3 tier-result scenarios

---

## Phase 2. LM1 Nurture + LM2 MVP (Weeks 5–7)

### Scope

Two parallel workstreams: build out the LM1 email nurture sequences for all 3 tiers, AND ship the LM2 Process Map.

### Week 5. LM1 nurture sequences

- [ ] Set up Tier A nurture in the email tool (5 emails from `04-readiness-filter-emails.md`)
- [ ] Set up Tier B nurture (6 emails)
- [ ] Set up Tier C nurture (welcome + bi-weekly rotation of 10 topics)
- [ ] Wire Make.com to enroll new contacts in the correct sequence based on `scoring.tier`
- [ ] Test enrollment: submit one fake response per tier, confirm correct sequence triggers
- [ ] Set the BSS calendar link on Tier A emails to use UTM tracking so we can measure conversion

### Week 6. LM2 build

- [ ] Build `/orlando-homebuying-roadmap` standalone landing page from `05-process-map-spec.md`
- [ ] Build `/orlando-homebuying-roadmap/get` opt-in form (prefill logic for `source=lm1_tier_b`)
- [ ] Build `/orlando-homebuying-roadmap/view` rendered roadmap from `06-process-map-content.md`
- [ ] Design and produce the static English PDF version (Canva)
- [ ] Wire Make.com webhook to handle `magnet: "lm2"` payloads, including the `lm1_tier_b` routing rule (pause LM1 Tier B nurture, start LM2 nurture)
- [ ] Build LM2 transactional email with PDF attachment

### Week 7. LM2 nurture + integration testing

- [ ] Set up LM2 nurture sequence (3 emails from `07-process-map-emails.md`)
- [ ] Update Tier B result page CTA to link to the new LM2 prefilled opt-in
- [ ] Update Tier A result CTA copy to mention "if you want the roadmap too, it's here"
- [ ] End-to-end integration test:
  - Submit LM1 as Tier B → receive Tier B email → click CTA → opt into LM2 → receive LM2 email + roadmap → confirm LM1 Tier B sequence is paused
  - Submit LM2 as standalone → receive LM2 email → confirm new contact tagged correctly
- [ ] Monitor inbox for any reply that says "español". Manually respond per Phase 3 plan

### Definition of Done for Phase 2

- All 3 LM1 nurture sequences send on schedule for fake submissions in each tier
- LM2 opt-in works from both entry points
- A Tier B contact does not receive duplicate emails when they opt into LM2
- All emails have correct merge tags (no `{{first_name}}` showing up as literal text)
- Agent has reviewed every email in every sequence in a real preview send

---

## Phase 3. Spanish + Polish (Weeks 8–10)

### Scope

Add the bilingual edge that is the agent's specific competitive advantage in the Orlando market. Plus operational polish.

### Week 8. Spanish content

- [ ] Translate all LM1 quiz screens, result page tier content (Tier A / B / C), and market snapshot (professional translation, not just Google Translate)
- [ ] Translate all LM2 roadmap content
- [ ] Translate all transactional emails and nurture emails
- [ ] Produce the Spanish PDF version of the roadmap
- [ ] Add a language toggle on the LM1 landing page (English / Español)
- [ ] Add `/orlando-homebuying-readiness-quiz/start?lang=es`, `/orlando-homebuying-readiness-quiz/result?lang=es`, `/orlando-homebuying-roadmap/view/es`
- [ ] Update Make.com to send Spanish emails when `preferred_language = "es"`

### Week 9. Monthly market-update list

- [ ] Set up the long-term monthly nurture list in the email tool
- [ ] Route all "graduates" from Tier A, B, C, and LM2 nurture sequences into this list
- [ ] Draft the first 3 monthly emails (so the agent isn't writing them under deadline)
- [ ] Set a scheduled task reminder for the 1st of each month to write the next one

### Week 10. Analytics + iteration setup

- [ ] Build a simple dashboard (or Google Sheet) tracking:
  - Daily LM1 starts vs. completes
  - Tier distribution by week
  - LM2 opt-in rate from Tier B
  - BSS booking rate from Tier A
  - Email open + click rates per sequence
- [ ] Identify the lowest-converting step (likely the email gate) and queue an A/B test
- [ ] Set up a scheduled task: weekly summary of funnel metrics to the agent's email

### Definition of Done for Phase 3

- A Spanish-only speaker can complete the entire LM1 + LM2 flow without ever seeing English
- The agent receives a weekly funnel-metrics email
- The monthly market-update list has 3 emails pre-drafted

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

| Risk | Mitigation |
|---|---|
| Make.com hits its operations limit on a free plan | Phase 2 onward, upgrade to the Pro plan ($16/mo as of 2026). Forecast 30 ops per submission. |
| Email tool flags the agent's domain for low engagement | Warm up the sending domain by sending the first 50 emails to a list of friends/family who will open. After that, organic volume protects reputation. |
| Translation quality is poor and burns Spanish-speaking trust | Pay for a professional translator who is native to Florida Latino markets (Cuban or Puerto Rican Spanish, not Castilian). Have one Spanish-speaking client review before launch. |
| Agent runs out of evening hours to ship | Phases 1, 2, 3 can each slip by 2 weeks without breaking the funnel. Don't shortcut quality on the questions, scoring logic, or result-page copy; those are the asset. |
| Make.com webhook fails silently and submissions get lost | Add a fallback: every webhook also writes to a Google Sheet. If the email send fails, the agent can manually recover within 24 hours. |

---

## What we're explicitly not building

These came up during planning and were deferred. Documenting so they don't get re-litigated mid-build.

- A self-hosted CRM (Rails or otherwise). The email tool's contact view is enough for Year 1.
- A custom-built calendar tool. Cal.com or Calendly handles BSS booking. Building one is a distraction.
- A buyer's portal where prospects can log in and "see their progress." This sounds great and adds nothing to conversion in Year 1.
- Native iOS / Android apps. The funnel is web-only. The agent's iOS skills are *not* in scope for this project.
- Anything paid (ads, lead lists, etc.). Hard rule from the project knowledge base.
