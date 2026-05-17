## What this doc is

This is the operating reference for the **Live Math** and the **3-Neighborhood Shortlist**, the two deliverables that carry most of the value in a Buyer Strategy Session. The high-level offer and the call structure live in [09-bss-offer-spec.md](09-bss-offer-spec.md); this doc covers the numbers, the formulas, the data sources, and the decision logic the agent runs in real time during the call and during the 24-hour post-call window.

Status: **DRAFT**. The Orlando-specific defaults (tax rates, insurance ranges, starting price-per-square-foot bands by ZIP) are calibration starting points; replace them with measured values once the agent has booked the first 10 prospects.

---

## v0 MVP cut — what from this doc ships first

Per the v0 scope in [09-bss-offer-spec.md](09-bss-offer-spec.md), this doc is preserved almost entirely intact in v0. The math sheet (Google Sheet) and the Shortlist PDF (Google Doc exported to PDF) are the deliverables that carry the BSS's perceived value. They ship as designed.

What is **deferred** from this doc to v1:

- The `BSS_MATH_CONFIG` TypeScript object near the end of the doc. v0 has no code; the calibration values live in the math sheet's cell defaults and in this doc.
- A JavaScript-calculator version of the math sheet on the BSS landing page (implied by the "re-host the same logic in JavaScript" note in the Formulas section). v0 only has the live-on-call Google Sheet. A self-serve calculator widget is a v1+ project, gated on whether the BSS funnel produces enough volume to justify pre-call math exploration.
- Automated Stellar MLS pulls into the Shortlist PDF build pipeline. In v0 the agent pulls comps manually the morning the PDF is built.

Everything else in this doc, including the math sheet tab structure, formulas, shortlist branching logic (Paths A, B, and C), the Shortlist PDF page structure, and the Florida-specific data defaults, ships in v0.

---

## How the math sheet is organized

The math is a Google Sheet, one tab template that the agent duplicates per prospect at booking time. The sheet must be designed so that the agent can talk while editing: cells the agent types into are obvious, cells that calculate are protected, and the result that matters (the monthly all-in payment + total cash to close) is large enough to read on a shared screen.

### Tabs

| Tab | Job |
|---|---|
| **Inputs** | Down payment, credit band, target price, term, county, property type, HOA, expected rate. This is where the agent types. |
| **Loan Scenarios** | Side-by-side calculation of three loan paths: FHA 3.5% down, Conventional 5% down, Conventional 20% down (if affordable). Each path computes loan amount, monthly P&I, MIP or PMI, and adds the housing components below. |
| **PITI Build-Up** | Property tax (county and homestead-aware), Florida HO3 insurance, HOA. Pulled by formula into Loan Scenarios. |
| **Cash to Close** | Down payment + closing costs (lender, title, prepaids, escrow setup, recording, doc stamps, intangible tax). |
| **Reserves After Close** | What the prospect has left after closing. The single highest-impact number for a Tier A first-time buyer to see, because it answers "will I be house-poor?" |
| **Shortlist Math** | Three named ZIP rows. Each row pulls the same payment build-up against a different median price for that ZIP and shows the gap (in dollars and in months of saving) between the prospect's number and the ZIP's number. |

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
| B14 | Neighborhoods the prospect named (free text) | "Sanford, maybe Lake Mary" |

Cell B14 drives the Shortlist Math tab; see "Shortlist decision logic" below.

---

## Formulas (the actual math)

The math sheet is mechanical. The agent does not derive these on the call; the cells already compute them. They are documented here so the sheet can be rebuilt from scratch if it gets corrupted, and so a future engineer can re-host the same logic in JavaScript on the BSS landing page if the call ever moves to a calculator widget.

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

Conventional 3% (HomeReady / Home Possible) is available as a switch on the Inputs tab for prospects with strong credit and very thin cash. Hidden by default to avoid cluttering the call.

### Mortgage insurance

| Scenario | Upfront | Monthly |
|---|---|---|
| FHA 3.5% | 1.75% of loan amount, financed into the loan | `loan * 0.0055 / 12` (annual MIP 0.55% as of 2026; verify against current FHA HUD handbook before each call) |
| Conventional < 20% | none | `loan * 0.005 to 0.015 / 12` depending on credit and LTV; use 0.6% as default for 680-739 / 95% LTV until a real quote replaces it |
| Conventional 20% | none | none |

Mortgage insurance is the single most-mis-estimated line item by first-time buyers; if the prospect has been quoted "your monthly is around X" by a builder's sales office, the chances are even that X excludes MI. The math sheet shows it on its own line, never bundled, so the prospect sees it.

### Florida property tax

```
Annual tax = (assessed_value - homestead_exemption) * millage_rate
```

Starting estimates by county along the Sanford-Downtown Orlando corridor:

| County | Millage estimate | Notes |
|---|---|---|
| Seminole | ~14.5 mills (1.45%) | Higher inside city limits (Sanford, Lake Mary). |
| Orange | ~13.5 mills (1.35%) | Downtown Orlando city add-on. |

Homestead exemption is $50,000 for primary residences. The **Save Our Homes** cap limits annual assessed-value increases to 3% or CPI, whichever is lower, so the first-year tax bill on a just-purchased home can be meaningfully higher than what the seller paid the year before. **Always quote the new-owner number, not the prior tax bill.** Florida county property appraiser sites have a "tax estimator" tool keyed to sale price; use it during the call when the price is non-default.

### Florida homeowners insurance (HO3)

Florida insurance is currently the second-largest source of payment surprise after MI. As of 2026, in the Sanford↔Downtown Orlando footprint, expect:

| Property profile | Annual HO3 estimate |
|---|---|
| SFH, 2000s or newer construction, 30+ miles inland | $2,800 to $3,800 |
| SFH, 1980s or older, older roof, inland | $3,800 to $5,500 |
| Townhome, master policy covers exterior | $1,200 to $2,000 (HO6) |
| Condo, master policy strong | $900 to $1,400 (HO6) |

Refresh these bands quarterly. If the prospect is buying in a flood zone AE or VE (relevant for some Sanford properties near the St. Johns River) add a flood policy: $700 to $2,500/year depending on elevation certificate.

### Cash to close

```
Cash to close = down payment + closing costs + prepaids + escrow setup
```

Closing-cost starting estimate for a $360k purchase in this corridor: $7,500 to $10,500 before seller credits. Components:

- Lender fees (origination, underwriting, processing): $1,500 to $2,500
- Title (owner + lender policy, settlement): $2,000 to $3,500
- Recording + doc stamps + intangible tax: $1,200 to $2,000 (Florida-specific)
- Prepaids (first-year insurance, 2-3 months tax escrow, prepaid interest): $1,500 to $3,000
- Inspection + appraisal (paid pre-close, sometimes after deposit): $700 to $1,200

The math sheet keeps these as line items so the agent can show what's negotiable (seller credit toward closing costs) and what is not (doc stamps).

### Reserves after close

```
Reserves = cash_available - cash_to_close
```

A Tier A first-time buyer with under 1 month of reserves after close is a yellow flag; under 0.5 months is a red flag the agent surfaces in the next-step block. The math sheet flags this automatically with a color band.

### Front-end and back-end DTI

```
Front-end DTI = total_PITI / gross_monthly_income
Back-end DTI = (total_PITI + other_monthly_debts) / gross_monthly_income
```

The math sheet shows both. FHA tolerates back-end DTI up to ~57% in some cases; conventional typically caps at ~45%. If the prospect's back-end DTI runs over 43%, the math sheet color-bands it so the conversation can shift to either a lower price target or a longer pre-approval timeline.

---

## Shortlist decision logic

The shortlist is **not** a fixed deliverable shape. It branches on whether the prospect walked in with a neighborhood preference. The branch happens during the diagnostic block of the call (see the run-of-show in [09-bss-offer-spec.md](09-bss-offer-spec.md)), not pre-call. The agent's question is verbatim:

> *"Before I show you neighborhoods, two ways we can do this. Either you tell me where you're already looking and I'll run the math on those places first, or I show you three I'd pick from your numbers and you tell me what's missing. Which one?"*

Then one of three paths runs.

### Path A: Prospect has no neighborhoods in mind

The agent picks three ZIPs along the Sanford↔Downtown Orlando corridor that fit the prospect's math, household, and stated must-haves. The pick is not random; it's a deliberate spread:

1. **The anchor.** The ZIP where the prospect's number is most comfortable, with room for reserves. Default to the higher-affordability end of the corridor (Sanford, Apopka, Casselberry) for prospects with thin cash or back-end DTI tight against limits.
2. **The stretch.** A ZIP that costs more but unlocks something specific the prospect named in the intake (commute, schools, walkability). The agent shows what it costs in payment terms, not in price terms. ("Maitland is $480 more a month, every month, for 30 years.")
3. **The wildcard.** A ZIP the prospect probably has not considered but that fits their numbers better than they would guess. This is the highest-value pick because it reframes the search. Eatonville, College Park, or Casselberry often serve here.

### Path B: Prospect has 1 or 2 neighborhoods in mind, and the math works

The agent runs the math on the prospect's neighborhoods first, in real time, in front of them. Then the shortlist becomes:

1. **Their first named ZIP.** Math validated. The payment, the cash to close, the reserves remaining.
2. **Their second named ZIP, or an adjacent one the agent recommends.** Same math, same lens.
3. **One agent-picked alternate.** Same rules as the "wildcard" in Path A: something they did not consider but that fits. The agent introduces it explicitly: *"You did not ask for this one, so feel free to ignore it, but here's the case for adding it to your list."*

The prospect's preference always leads. The shortlist PDF orders the named ZIPs first and the agent's pick last; never inverted.

### Path C: Prospect has neighborhoods in mind, and the math does not work

This is the path that requires the most care. The prospect named a place they cannot afford at their current numbers. The agent's job is to be honest without crushing the dream. The script:

> *"Here's the math on Winter Park at your current cash and rate. The payment lands at $X, your reserves after close are negative, and your back-end DTI runs at Y. I can show you three things that change this picture: either you wait six months and grow the cash side, you stretch to FHA at a smaller home in the same ZIP, or you keep the price and slide one ZIP outward where the same square footage costs $Z less. Want me to walk through all three?"*

Then the shortlist PDF contains:

1. **The named ZIP, with the gap surfaced.** Not buried, not avoided. Payment, reserves, DTI, all on the page. This is the "what it would actually cost" page.
2. **The compromise inside the named ZIP.** Smaller home, older home, townhome instead of SFH, FHA instead of conventional. Whichever lever buys the most affordability without changing the location.
3. **The honest alternate.** One ZIP outward, same product type, fits the math comfortably. Named with the specific trade-off (commute minutes, school zone, drive to whichever anchor the prospect cares about).

**The shortlist is never blank, and it never says "you cannot afford this."** It always shows the path. The prospect leaves with three live options even when their dream ZIP is out of reach. That is the deal.

### Adding a fourth ZIP

The shortlist is three by default but the agent may add a fourth when:

- The prospect named two ZIPs that are very close together (Lake Mary + Longwood) and a meaningful third agent-pick exists.
- The prospect's must-have eliminates everything in the first three (e.g., "must have boat access to the St. Johns" leaves Sanford and almost nothing else; the fourth is the honest second-best).
- The math meaningfully shifts between two scenarios (FHA vs conventional) and the agent wants to show the prospect what their cash side unlocks in a different ZIP.

Never add a fifth. Five ZIPs is a Zillow tab, not a shortlist.

---

## What goes into the Shortlist PDF

The Shortlist PDF is the post-call deliverable. It is **personalized**, produced from the math sheet and the call notes, and sent within 24 hours of the call ending. Structure:

### Cover

- Prospect's first name + co-buyer if present.
- The agent's voice line at the top: *"Here's what we talked about, written down so you can use it whether or not we ever work together."*
- The math summary in one box: target price, monthly all-in payment, cash to close, reserves after close.

### Page per ZIP

One page per ZIP on the shortlist. Each page contains:

- ZIP code, neighborhood label (the human name, not just numbers), county.
- Why this ZIP for this prospect, in two or three sentences. References what they said on the call.
- The math at this ZIP: median sale price in the prospect's product type, what the monthly payment becomes, the gap (positive or negative) vs the prospect's number.
- One **current** comp under contract or recently sold in this ZIP, in the prospect's product type and price band. Address, beds/baths/sqft, list price, status. Pulled from Stellar MLS the morning the PDF is built.
- The trade-off in plain English. ("Sanford gives you 200 more sqft for the same dollar. The commute to Maitland is real. Schools are mixed; if school zoning matters, walk the specific street before bidding.")
- One Orlando-specific gotcha for this ZIP. (Septic-to-sewer zones in older Sanford streets, flood zone AE near the St. Johns, HOA delinquency in some 2000s Apopka developments, etc.)

### Closing page

- The three options from the call's close, written down: shortlist only, lender intro, buyer's agent engagement.
- The agent's contact, license number, brokerage.
- A reminder that the math sheet link is live and editable so the prospect can re-run with different numbers as their cash grows or the rate moves.

### What the PDF does not contain

- The prospect's full LM1 answers. Those live in Pipedrive; the PDF is forward-looking, not a recap of the score.
- A booking link or a CTA pushing the buyer's agreement. The post-call email carries that; the PDF stays a reference document.
- Speculative comps or aspirational pricing. Every comp on the PDF is real and dated. If the agent cannot find a real comp for a ZIP-product combination, that ZIP does not belong on the shortlist.

---

## Data sources and refresh cadence

| Data | Source | Refresh |
|---|---|---|
| Median sale price by ZIP and product type | Stellar MLS, sales last 90 days | Re-pull morning of PDF build |
| Active and pending comps | Stellar MLS, status pending/active under contract | Same |
| Property tax millage | County property appraiser site (Seminole, Orange) | Quarterly; check for millage changes after county budget season (August/September) |
| Insurance ranges | Local insurance broker quarterly check-in | Quarterly |
| FHA MIP rates | HUD handbook 4000.1 | Annually, and whenever the agent hears FHA changes in the news |
| Conventional PMI rates | A real lender quote against a recent file | Refresh whenever the lender card is refreshed |
| Today's rate | The lender on the 3-lender card the agent expects to introduce | Day of call |

If any of these are stale during a call, the agent says so out loud: *"That insurance number is from last quarter; we'll plug in your real quote when you talk to the lender."* Visible honesty here protects perceived likelihood; covering it up does the opposite.

---

## What lives in `src/config/bss.ts` for the math

Per the [CLAUDE.md](../CLAUDE.md) configuration principle. None of these are inline-hardcoded once a JavaScript calculator version of the math sheet exists.

```ts
export const BSS_MATH_CONFIG = {
  termYearsDefault: 30,
  fhaDownPaymentPct: 0.035,
  fhaUpfrontMipPct: 0.0175,
  fhaAnnualMipPct: 0.0055,        // 2026 default; verify against HUD 4000.1
  conventionalLowDownPct: 0.05,
  conventionalLowDownPmiPct: 0.006, // calibrate to credit band
  conventionalFullDownPct: 0.20,
  homesteadExemption: 50000,
  millageByCounty: {
    seminole: 0.0145,
    orange: 0.0135,
  },
  insuranceAnnualRanges: {
    sfhNewer: [2800, 3800],
    sfhOlder: [3800, 5500],
    townhome: [1200, 2000],
    condo: [900, 1400],
  },
  reservesYellowMonths: 1,
  reservesRedMonths: 0.5,
  dtiBackEndYellow: 0.43,
  dtiBackEndRed: 0.50,
} as const;
```

Everything in this object is a calibration target. None of them are structural constants. The whole point of putting them in config is so the agent can update tax millage in October without touching code.

---

## Hard rules for the math

Inherited from [CLAUDE.md](../CLAUDE.md) and made specific to the BSS math.

1. **Never generate a number you have not verified.** This includes rates, MI percentages, tax millage, and insurance bands. The agent's perceived-likelihood collapse comes from confidently quoting a wrong payment, not from saying "I'll confirm that."
2. **Never bundle MI into P&I in any deliverable.** The line item must always be visible. Bundling is how builder-preferred-lender quotes mislead first-time buyers.
3. **Use the new-owner tax number, not the seller's last tax bill.** Save Our Homes resets on sale and most first-time buyers do not know this.
4. **The math sheet link goes to the prospect at the end of the call, every call.** Even no-shows who reschedule get the link once they show. The math is not a closed-source pitch deck.
5. **The shortlist always leads with the prospect's preference if they had one.** Never reorder to push the agent's pick. The prospect's preference is the input; the math is the diagnostic.

---

## Open questions to resolve before locking

- **The conventional 3% switch.** Conventional 3% products (HomeReady, Home Possible) are real and matter for Tier A buyers with strong credit and thin cash. Decide whether the math sheet shows them as a fourth scenario by default or keeps them on a hidden toggle that the agent flips when relevant.
- **Buyer agent compensation in the math.** Post-NAR-settlement, Florida buyer's agent compensation is sometimes paid by the buyer, sometimes by the seller, sometimes split. The cash-to-close line should have an explicit row for it. Pick a default assumption (seller-paid, buyer-paid 2.5%, etc.) and make the cell editable.
- **HOA estimate source.** Estimating HOA pre-offer is fuzzy; the seller's disclosure is the source of truth but it does not exist at the BSS stage. Decide whether the math sheet uses neighborhood averages or whether the agent enters a single representative value the prospect can override.
- **The 24-hour SLA on the Shortlist PDF.** This is currently the default in [09-bss-offer-spec.md](09-bss-offer-spec.md). If the agent can credibly hit 4 hours and the math sheet is already filled in live, decide whether to advertise the faster turnaround on the BSS landing page. Compresses the time-delay term.

---

## Related documents

- [09-bss-offer-spec.md](09-bss-offer-spec.md) — The BSS offer, run-of-show, Pipedrive fields, and call routing
- [10-bss-content.md](10-bss-content.md) — Landing page, intake form, in-call script, deliverable templates
- [11-bss-emails.md](11-bss-emails.md) — Pre-call, post-call, and no-show email drafts
- [08-implementation-roadmap.md](08-implementation-roadmap.md) — When the math sheet, the Shortlist PDF template, and the calendar integration ship
- [CLAUDE.md](../CLAUDE.md) — Funnel namespace, hard rules, market context, numbers-must-be-validated rule
