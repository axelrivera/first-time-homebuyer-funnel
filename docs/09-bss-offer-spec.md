
This is the offer spec for the **Buyer Strategy Session (BSS)**, the 30-minute conversation that terminates both LM1 (Tier A) and LM2. The BSS is the conversion conversation in the funnel. LM1 and LM2 did the giving. The BSS is where the agent finds out if there is a fit and asks for the relationship.

Status: **DRAFT**. This spec is deliberately small. A solo agent runs the BSS by hand, end to end, with no automation beyond a calendar booking link and three Pipedrive fields.

---

## What the BSS is

A 30-minute one-on-one conversation between the agent and a prospect who has already qualified by completing LM1 as Tier A, or by completing LM2 (which means they were Tier B and chose to keep going).

The BSS is **a conversion conversation, not a free consulting session**. LM1 and LM2 already gave the prospect substantial value: a readiness score with personalized tier guidance and, for Tier B, a step-by-step roadmap. The BSS is where the agent learns more about their specific situation, answers their questions, and proposes one of three concrete next steps.

Channel: video call (Google Meet link in the Google Calendar invite). Phone is fine if the prospect requests it. The agent does not push for in-person at this stage.

Language: English only for now. Spanish-language delivery is deferred until the English funnel proves out.

---

## How it gets booked

The BSS has no public landing page. The booking surface is the agent's existing **Google Calendar appointment scheduling page**. The booking link is placed on these surfaces only:

- The LM1 Tier A result page
- Tier A nurture emails A1 through A5
- The LM2 viewer page
- LM2 nurture emails

It is **not** placed on the homepage, in nav, in the LM1 Tier B or Tier C result pages, in any cold-outreach DM, or in any paid surface.

Google Calendar handles the booking confirmation and the reminder email natively. No custom landing page, no intake form, no Make.com integration with Calendar.

---

## Eligibility

**Tier A** prospects (from LM1) and **LM2 graduates** (originally Tier B, completed LM2) are the only sources. Both have already opted in, completed at least one diagnostic, and received nurture. The agent has their LM1 answers in Pipedrive before the call.

**Tier C is never offered the BSS.** Enforced by not placing the booking link on any Tier C surface. If a Tier C contact books somehow (forwarded link), the agent decides manually whether to honor the booking or redirect to nurture.

Re-takes change tier. If a prospect re-takes LM1 and moves from C to B or A, the new tier governs.

---

## 30-minute structure

Three blocks. Directional, not scripted. The agent flexes the order based on what the prospect brings.

| Block | Minutes | What happens |
|---|---|---|
| **Recap and reset** | 0:00 to 5:00 | Agent reads back what they already know from the prospect's LM1 (and LM2, if applicable) answers in one or two sentences. Asks: *"What has changed since you took the quiz?"* |
| **Diagnose and answer** | 5:00 to 20:00 | The prospect's questions, plus the agent's diagnostic questions: timeline, cash available, neighborhoods being considered, lender status (pre-approved, talking to one, none yet), deal-breakers. The agent answers anything specific the prospect raises. The worksheet from [12-bss-math-and-shortlist.md](12-bss-math-and-shortlist.md) is open on a second tab if a number question comes up. The worksheet is the agent's internal tool; it is not shared. |
| **Next step** | 20:00 to 30:00 | The agent proposes one of three next steps and asks for a decision: (1) Sign the Buyer Brokerage Agreement (BBA) and start the search. (2) Take a lender introduction; revisit in two weeks. (3) Stay in nurture; revisit in 30 to 60 days. |

The agent walks into every BSS with the prospect's Pipedrive record open. **Stop at 30 minutes** unless the prospect explicitly says "keep going." Respecting the cap is part of the offer.

---

## Outcome recording

Within 1 hour of the call ending, the agent sets `fthb_bss_outcome` in Pipedrive to one of:

| Value | Means |
|---|---|
| `signed` | Prospect signed the BBA on the call or committed in writing within 24 hours |
| `not_yet` | Prospect attended, conversation was useful, but no commitment today. Could be a lender intro, "we'll think about it," or "let's talk again in a few weeks." Agent's call notes capture the specifics. |
| `no_show` | Prospect did not attend and did not reschedule before the slot started |

That is the entire enum. The nuance (which "not yet" sub-flavor, whether a lender intro happened, when to revisit) lives in the agent's Pipedrive notes, not in an enum value.

---

## After the call

**If `signed`**: the agent moves to the active-client onboarding flow (out of scope for this doc).

**If `not_yet`**: the agent sends an optional plaintext follow-up note within 24 hours if the conversation warranted one. Template in [11-bss-emails.md](11-bss-emails.md). A lender introduction, if offered, is a separate plaintext email cc'ing the lender. The contact returns to whichever nurture campaign they were paused from (LM1 Tier A, LM2 nurture). The agent decides manually whether to set a Pipedrive follow-up reminder for themselves.

**If `no_show`**: the agent decides whether to email a reschedule link or let it close. No automated no-show sequence.

---

## Pipedrive custom fields

Three fields. All set manually by the agent.

| Field | Type | Notes |
|---|---|---|
| `fthb_bss_booked_at` | datetime | Set on booking. Agent copies from the Google Calendar event when entering it. |
| `fthb_bss_attended` | bool | Set within 1 hour of the call. False means no-show. |
| `fthb_bss_outcome` | enum | `signed` / `not_yet` / `no_show`. Set within 1 hour of the call. |

All field names use the `fthb_` cross-system prefix per [CLAUDE.md](../CLAUDE.md)'s namespace convention.

---

## Analytics

One event, fired client-side from the booking buttons in Tier A emails, the LM1 Tier A result page, and the LM2 surfaces:

| Event | When |
|---|---|
| `fthb_bss_cta_click` | The "book a call" button is clicked, regardless of which surface |

Bookings are counted from Google Calendar; attendance and outcomes from Pipedrive. No automated funnel-stage event pipeline.

---

## Voice and copy rules

All BSS-facing copy follows the project rules from [CLAUDE.md](../CLAUDE.md). Specifically:

- **No value-stack framing on the BSS itself.** The BSS is not a Grand Slam Offer with bonuses. It is a call. The funnel already gave; this is the ask.
- **Do not promise deliverables.** No personalized shortlist PDF, no lender comparison card, no math sheet share link, no red-flag filter. If a specific prospect would benefit from one, send it as a one-off. Do not promise it on the booking surface or in nurture emails.
- **Soft framing on the way in, direct posture on the call.** The CTA wording across all surfaces is "let's talk about your specific situation and figure out the next step." On the call, in Block 3, the agent asks for the decision plainly.

---

## Hard rules (do not violate)

1. **Tier C is never offered the BSS.** Enforced by surface placement: the booking link is not on Tier C result pages or in Tier C emails.
2. **No paid ads or boosted posts driving BSS bookings.** Organic content, LM1, LM2 only.
3. **No cold outreach to fill the calendar.** No list-buys, no scraping, no stranger DMs.
4. **No public landing page.** The booking link is reachable only from Tier A / LM2 surfaces, and from direct sharing by the agent in warm contexts (existing client referrals, sphere contacts).
5. **30 minutes means 30 minutes.** The cap is part of the deal. Running long once trains the calendar to expect it.

---

## What is intentionally out of scope

- The Buyer Brokerage Agreement (BBA) workflow once a prospect signs. That is the active-client onboarding spec.
- Spanish-language versions. Deferred to a later iteration.
- A self-serve math calculator on the site. The agent's worksheet is for the agent.
- Any automated email sequence around the BSS. Calendar handles confirmation and reminder. Anything else is a manual plaintext note.
- Make.com integration with Google Calendar. Manual Pipedrive entry is faster than building this at solo-agent volume.
- A personalized post-call deliverable (Shortlist PDF, lender comparison card, red-flag filter). The conversation is the deliverable.

---

## Related documents

- [01-strategy-and-funnel.md](01-strategy-and-funnel.md) — Why the BSS sits at the end of the funnel
- [02-readiness-filter-spec.md](02-readiness-filter-spec.md) — LM1 spec; Tier A is the primary BSS entry
- [03-readiness-filter-content.md](03-readiness-filter-content.md) — Tier A result page CTA
- [04-readiness-filter-emails.md](04-readiness-filter-emails.md) — Tier A emails A1 through A5 carry the BSS CTA
- [05-process-map-spec.md](05-process-map-spec.md) — LM2 spec; LM2 graduates are the secondary BSS entry
- [06-process-map-content.md](06-process-map-content.md) — LM2 viewer page CTA
- [07-process-map-emails.md](07-process-map-emails.md) — LM2 emails carry the BSS CTA for Tier B graduates
- [08-implementation-roadmap.md](08-implementation-roadmap.md) — Phased build plan
- [10-bss-content.md](10-bss-content.md) — In-call structure and CTA copy fragments
- [11-bss-emails.md](11-bss-emails.md) — The single optional plaintext follow-up template
- [12-bss-math-and-shortlist.md](12-bss-math-and-shortlist.md) — The agent's internal worksheet (not a deliverable; filename is vestigial)
- [CLAUDE.md](../CLAUDE.md) — Funnel namespace, hard rules, voice
