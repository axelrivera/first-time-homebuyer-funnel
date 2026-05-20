## What this doc is

The **agent's internal worksheet** for running quick payment and closing-cost math during a Buyer Strategy Session. The high-level offer lives in [09-bss-offer-spec.md](09-bss-offer-spec.md); the call structure that this worksheet supports lives in [10-bss-content.md](10-bss-content.md).

This is **not** a deliverable. The worksheet is a Google Sheet the agent keeps on a second monitor or browser tab during the call. The prospect never sees it (unless the agent chooses to screen-share in the moment to answer a specific number question) and the prospect does not receive a copy after the call.

Status: **DRAFT**. The Orlando-specific defaults (tax rates, insurance ranges) are calibration starting points; replace them with measured values once the agent has run the first 10 BSSes and knows which numbers prospects actually press on.

Filename note: this doc is named `12-bss-math-and-shortlist.md` for historical reasons. The "shortlist" half of the original concept has been removed (see "What used to live here" at the bottom). The filename will be renamed in a later sweep; the cross-references in the other BSS docs still point here.

---

## How the worksheet is organized

A single Google Sheet, one tab template that the agent **duplicates per prospect** at booking time and pre-fills with what the LM1 (and LM2, if applicable) answers tell them. The sheet is built so the agent can talk while computing: the inputs are obvious, the outputs are large enough to read, and the conditional flags (yellow or red on reserves and DTI) are visible at a glance.

### Tabs

| Tab | Job |
|---|---|
| **Inputs** | Down payment, credit band, target price, term, county, property type, HOA, expected rate. |
| **Loan Scenarios** | Side-by-side calculation of three loan paths: FHA 3.5% down, Conventional 5% down, Conventional 20% down. Each computes loan amount, P&I, mortgage insurance, plus the housing components below. |
| **PITI Build-Up** | Property tax (county- and homestead-aware), Florida HO3 insurance, HOA. Pulled by formula into Loan Scenarios. |
| **Cash to Close** | Down payment + closing costs (lender, title, prepaids, escrow setup, recording, doc stamps, intangible tax). |
| **Reserves After Close** | What the prospect has left after closing. The single highest-leverage number to be able to quote on the call. |

There is no "Shortlist Math" tab. The reframed BSS does not produce a personalized shortlist deliverable, so the dedicated tab that fed the Shortlist PDF has been removed.

### Cells the agent types into (Inputs tab)

| Cell | Label | Example |
|---|---|---|
| B2 | First name | Maria |
| B3 | Co-buyer? | No |
| B4 | Combined gross income, monthly | $7,800 |
| B5 | Existing monthly debts (cars, students, minimums) | $620 |
| B6 | Credit band (per LM1 Q1) | 680-739 |
| B7 | Cash available for the deal | $32,000 |
| B8 | Target purchase price (starting guess) | $360,000 |
| B9 | Term (years) | 30 |
| B10 | Property type | SFH |
| B11 | County | Seminole |
| B12 | Estimated HOA, monthly | $0 |
| B13 | Expected rate today | 6.875% |

---

## Formulas (the actual math)

The worksheet is mechanical. The agent does not derive these on the call; the cells compute them. Documented here so the sheet can be rebuilt from scratch if it gets corrupted.

### Monthly principal and interest

```
P&I = L * (r * (1+r)^n) / ((1+r)^n - 1)
```

Where:
- **L** = loan amount = purchase price minus down payment
- **r** = monthly interest rate = annual rate / 12
- **n** = number of monthly payments = term in years * 12

### Loan amount per scenario

| Scenario | Down payment | Loan amount |
|---|---|---|
| FHA 3.5% | `price * 0.035` | `price * 0.965` |
| Conventional 5% | `price * 0.05` | `price * 0.95` |
| Conventional 20% | `price * 0.20` | `price * 0.80` |

Conventional 3% (HomeReady, Home Possible) is available as a hidden toggle on the Inputs tab for prospects with strong credit and very thin cash. Hidden by default to keep the call moving.

### Mortgage insurance

| Scenario | Upfront | Monthly |
|---|---|---|
| FHA 3.5% | 1.75% of loan amount, financed into the loan | `loan * 0.0055 / 12` (annual MIP 0.55% as of 2026; verify against current FHA HUD handbook) |
| Conventional under 20% | none | `loan * 0.006 / 12` default for 680-739 / 95% LTV; replace with a real lender quote when one is available |
| Conventional 20% | none | none |

Mortgage insurance is shown on its own line, never bundled into P&I. If a prospect was quoted "your monthly is around X" by a builder's preferred lender and X excluded MI, the worksheet exposes that gap immediately.

### Florida property tax

```
Annual tax = (assessed_value - homestead_exemption) * millage_rate
```

Starting estimates by county across the Orlando metro (the two counties the buyer-facing market report spans):

| County | Millage estimate | Notes |
|---|---|---|
| Seminole | ~14.5 mills (1.45%) | Higher inside city limits (Sanford, Lake Mary, Longwood, Winter Springs, Oviedo). |
| Orange | ~13.5 mills (1.35%) | Downtown Orlando city add-on. Apopka, Maitland, Winter Garden, and Lake Nona each carry their own city/CDD adjustments — confirm per address. |

Homestead exemption is $50,000 for primary residences. **Save Our Homes** caps annual assessed-value increases to 3% or CPI, whichever is lower, so the first-year tax bill on a just-purchased home can be meaningfully higher than what the seller paid the year before. Always quote the new-owner number, not the prior tax bill. The county property appraiser sites have tax estimator tools keyed to sale price; use them on the call when the price moves off the default.

### Florida homeowners insurance (HO3)

Florida insurance is currently the second-largest source of payment surprise after mortgage insurance. As of 2026, across the Orlando-metro footprint the market report covers, expect:

| Property profile | Annual HO3 estimate |
|---|---|
| SFH, 2000s or newer construction, 30+ miles inland | $2,800 to $3,800 |
| SFH, 1980s or older, older roof, inland | $3,800 to $5,500 |
| Townhome, master policy covers exterior | $1,200 to $2,000 (HO6) |
| Condo, master policy strong | $900 to $1,400 (HO6) |

Refresh these bands quarterly. If the property is in flood zone AE or VE (relevant for some Sanford properties near the St. Johns River), add a flood policy: $700 to $2,500 per year depending on elevation certificate.

### Cash to close

```
Cash to close = down payment + closing costs + prepaids + escrow setup
```

Closing-cost starting estimate for a $360k purchase in this area: $7,500 to $10,500 before seller credits. Components:

- Lender fees (origination, underwriting, processing): $1,500 to $2,500
- Title (owner + lender policy, settlement): $2,000 to $3,500
- Recording + doc stamps + intangible tax: $1,200 to $2,000 (Florida-specific)
- Prepaids (first-year insurance, 2 to 3 months tax escrow, prepaid interest): $1,500 to $3,000
- Inspection + appraisal (paid pre-close): $700 to $1,200

The worksheet keeps these as line items so the agent can answer "what's negotiable" (seller credit toward closing costs) versus "what's not" (doc stamps).

### Reserves after close

```
Reserves = cash_available - cash_to_close
```

Under 1 month of reserves after close is a yellow flag; under 0.5 months is a red flag the agent surfaces in Block 3 of the call (the "next step" block). The worksheet color-bands this automatically.

### Front-end and back-end DTI

```
Front-end DTI = total_PITI / gross_monthly_income
Back-end DTI = (total_PITI + other_monthly_debts) / gross_monthly_income
```

FHA tolerates back-end DTI up to ~57% in some cases; conventional typically caps at ~45%. Back-end DTI over 43% is a yellow band on the worksheet; the conversation should shift to either a lower price target or a longer pre-approval timeline.

---

## How the worksheet is used during the call

The agent has the worksheet open on a second tab. The prospect does not see it unless a specific number question warrants a screen share. If the agent does share the screen, the goal is to answer the question and move on, not to make the worksheet itself part of the offer.

The most common moments where the worksheet earns its keep:

- **The prospect asks "what would my monthly payment look like?"** The agent has all the inputs from the LM1 / LM2 answers and can quote the FHA and Conventional 5% numbers within seconds.
- **The prospect has been quoted a number by a builder or a lender and wants a sanity check.** The agent runs their inputs into the same scenario and either confirms or surfaces what was excluded.
- **The prospect asks "can I afford X neighborhood?"** The agent plugs in the median price for that neighborhood and shows whether the payment, reserves, and DTI all clear.

The worksheet is the agent's diagnostic tool. It is not a deliverable. If the agent wants to follow up on a specific number after the call, the optional plaintext follow-up email in [11-bss-emails.md](11-bss-emails.md) is the surface.

---

## Data sources and refresh cadence

| Data | Source | Refresh |
|---|---|---|
| Property tax millage | County property appraiser sites (Seminole, Orange) | Quarterly; check after county budget season (August / September) |
| Insurance ranges | Local insurance broker quarterly check-in | Quarterly |
| FHA MIP rates | HUD handbook 4000.1 | Annually, and whenever the agent hears FHA changes in the news |
| Conventional PMI rates | A real lender quote against a recent file | When the lender bench refreshes |
| Today's rate | The lender the agent expects to introduce | Day of call |

If any of these are stale on a call, the agent says so out loud: *"That insurance number is from last quarter; we'll plug in your real quote when you talk to the lender."* Visible honesty here protects perceived likelihood; covering it up does the opposite.

---

## Hard rules for the math

Inherited from [CLAUDE.md](../CLAUDE.md):

1. **Never quote a number you have not verified.** Rates, MI percentages, tax millage, insurance bands. Confidently quoting a wrong payment is the fastest way to lose perceived likelihood.
2. **Never bundle mortgage insurance into P&I.** Always its own line item, on the worksheet and in any verbal quote.
3. **Use the new-owner tax number, not the seller's last bill.** Save Our Homes resets on sale; most first-time buyers do not know this.
4. **The worksheet is not shared by default.** If a specific number warrants follow-up, send it in the plaintext follow-up email per [11-bss-emails.md](11-bss-emails.md), not by sharing the sheet.

---

## What used to live here

Earlier drafts of this doc carried a "Shortlist decision logic" section (Paths A, B, C for whether the prospect walked in with neighborhoods in mind), a "What goes into the Shortlist PDF" template, a `BSS_MATH_CONFIG` TypeScript object for a future JavaScript calculator, and a Stellar MLS data-source section feeding the PDF build. Those have been removed. The reframed BSS does not produce a personalized shortlist deliverable; the agent talks through neighborhoods on the call and does not send a written ZIP-by-ZIP package afterward.

If the agent finds during the first 10 BSSes that a specific prospect would meaningfully benefit from a written neighborhood note, that is a one-off email, not a templated deliverable that gets promised on the booking surface.

---

## Open questions to resolve before locking

- **Conventional 3% switch.** Conventional 3% products (HomeReady, Home Possible) are real and relevant for Tier A buyers with strong credit and thin cash. Decide whether to surface as a fourth scenario column or keep on a hidden toggle.
- **Buyer agent compensation in the math.** Post-NAR-settlement, Florida buyer's agent compensation is sometimes paid by the buyer, sometimes by the seller, sometimes split. The cash-to-close line should have an explicit row. Pick a default assumption and make the cell editable.
- **HOA estimate source.** Seller's disclosure is the source of truth but does not exist at the BSS stage. Decide whether to use neighborhood averages or a single editable cell the agent overrides.
- **Filename.** Rename to `12-bss-worksheet.md` (or similar) in a later sweep that updates the cross-references in 09, 10, 11.

---

## Related documents

- [09-bss-offer-spec.md](09-bss-offer-spec.md) — Offer, outcome recording, hard rules
- [10-bss-content.md](10-bss-content.md) — In-call structure that this worksheet supports
- [11-bss-emails.md](11-bss-emails.md) — Optional plaintext follow-up email
- [CLAUDE.md](../CLAUDE.md) — Funnel namespace, hard rules, market context, numbers-must-be-validated rule
