
Fill in the three tier branches on the result page with the locked copy below. Each tier has: header + score badge, tier explanation paragraph, the "2 mistakes" section, and a tier-specific CTA.

## Prereqs

- [01-08 Result page shell + preview mode](./08-result-page-shell.md)

## Goal

The three `TierAContent`, `TierBContent`, `TierCContent` components (or branches inline in `result.astro`) render the locked copy below. Each tier ends with the right CTA, going to the right URL.

## Files to create / modify

```
src/components/result/TierAContent.astro
src/components/result/TierBContent.astro
src/components/result/TierCContent.astro
src/components/result/ResultFooter.astro
src/config/links.ts                                          (BSS booking URL constant)
src/pages/orlando-homebuying-readiness-quiz/result.astro     (swap stubs for real components)
```

## Voice anchor (applies to all three tiers)

The agent's voice: direct, educational, never salesy. First-person ("I"). No real-estate-agent clichés ("Now is a great time to buy!", "Let's find your dream home together!"). When in doubt, sound like someone explaining this to a younger sibling.

> **Voice anchor:** "Here's exactly what's standing between you and your first home, and what to do about it." Not: "Let's go on a journey together."

Block-quoted text below is **final copy — ship verbatim**. `{{display_score}}` is interpolated from the URL.

---

## Tier A: Ready Now (score 75–100)

### Header

> # You're in the "Ready Now" tier.
> ### Your score: {{display_score}} / 100
> Based on your answers, the realistic path from where you are today to keys in your hand is **30 to 60 days**. The work ahead isn't about getting ready. It's about not screwing up the last 5%.

### Tier explanation paragraph

> You answered like someone who has done the homework. Your credit, savings, and income are aligned, your timeline is tight, and you're not guessing about what you can afford. That's rare. Most people who land on this scorecard end up in the 90-Day Sprint tier or below. From here forward, deals fall apart for two reasons: avoidable financial mistakes between now and closing, and emotional decisions made in the heat of a competitive offer. The next section is built around both.

### The 2 mistakes Tier A buyers make most often

> ## The 2 mistakes you're most likely about to make
>
> **Mistake #1: Moving money or opening new credit between now and closing.**
> The day your lender pulls credit for pre-approval is *not* the last day they check. They re-pull right before closing. New car loan? Closing day disaster. Furniture financed on a store card "because we'll need it anyway"? Same. Even a large cash deposit your lender can't trace (like family help) can delay closing for 2 weeks while they verify the source. Rule: from pre-approval to keys, your financial life freezes. No new debt, no big deposits, no job changes, no closed accounts.
>
> **Mistake #2: Bringing a 2022 offer to a 2026 market.**
> The Orlando market you read about online is not the market you're buying in. Days on market are longer, price reductions are routine, and a buyer with real pre-approval and cash for closing has leverage back. Tier A buyers carry over an instinct from the last cycle: waive inspection, skip the appraisal contingency, write at or over list "to look serious." In this market, that instinct hands the seller free upside you didn't have to give. The right move is to keep all three contingencies (inspection, financing, appraisal). The inspection period is the broadest of the three: it's your unilateral window to walk for almost any reason and keep your deposit. Appraisal and financing are narrower triggers. Then start at or below list depending on days-on-market, and ask the seller for a closing-cost concession or rate buydown. If the seller says no, you still have the deal. You'll have spent zero negotiating leverage on a market that no longer demands it.

### CTA (Tier A)

> ## What's next
> You're ready. The next step is a 30-minute Buyer Strategy Session with me: free, no pitch. We'll go through your exact pre-approval, your two target neighborhoods, and the 3 offer mechanics that win in this market right now.
>
> [ Book the Buyer Strategy Session → ]
>
> *30 minutes. Zoom or in person.*

**Tier A CTA target:** the BSS booking URL. Define it once as `BSS_BOOKING_URL` in `src/config/links.ts`. Fire `fthb_readiness_cta_click` with `{ tier: 'A', cta_target: 'bss' }` on `pointerdown`.

---

## Tier B: 90-Day Sprint (score 45–74)

### Header

> # You're in the "90-Day Sprint" tier.
> ### Your score: {{display_score}} / 100
> Based on your answers, you're realistically **60 to 120 days** away from being able to make a strong, competitive offer on your first Orlando home. The runway is short enough to feel real and long enough that what you do in the next 90 days is the whole game.

### Tier explanation paragraph

> Most first-time buyers in Orlando land here, and there's a reason: you have most of the pieces, but one or two specific things (credit awareness, savings velocity, or pre-approval status) aren't fully in place yet. The good news is that the gap closes when you work at it deliberately. The bad news is that most buyers in this tier don't. They drift, get frustrated, and either rush into a bad offer or wait themselves out of the market. The next section is the two mistakes I see *this exact tier* make most.

### The 2 mistakes Tier B buyers make most often

> ## The 2 mistakes you're most likely about to make
>
> **Mistake #1: Shopping for homes before getting pre-approved.**
> This is the single biggest mistake in this tier. Scrolling Zillow and going to open houses *feels* like progress, but without a pre-approval letter in hand, you have zero negotiating power. Sellers in Orlando routinely throw out offers that don't include a pre-approval. Worse, you fall in love with a house before you know what you can actually afford, then either overstretch or get heartbroken. Rule: in this tier, you do **not** look at a single house in person until you have a pre-approval letter from a lender. Get the letter first. Then we shop.
>
> **Mistake #2: Dismissing new construction on list price alone.**
> The 2026 Orlando new-construction math has flipped. Builders are sitting on inventory and competing hard, which means lower mortgage rates and stacked incentives on new builds. It's common right now to see builder-subsidized rates dip below 6% while resale rates sit above 6%, with closing-cost concessions and finish packages layered on top. Run the numbers: a $340K new build at 5.49% (5% down, 30-year fixed) runs about $1,830/month in principal and interest. A $310K resale at 6.99% with the same down payment runs about $1,955/month. The list price is $30K higher; the P&I is $125/month *lower*. Taxes, HOA, and insurance often run higher on new construction, so the all-in picture can narrow or reverse that gap. The point is that the comparison is no longer obvious. Most Tier B buyers never run it. They shop resale because new construction "feels expensive," or because they absorbed prior-cycle advice warning them off builder lenders. The right move is to run the full monthly comparison (P&I + taxes + insurance + HOA) on a real new build alongside a real resale you'd actually consider, and to ask whether any advertised rate is permanent or a temporary buydown that resets in year 2 or 3. Pick the lane after the math, not before it.

### CTA (Tier B)

> ## What's next
> You've got a 90-day game plan ahead of you. Don't fly blind. Grab my **9-Step First Home Roadmap**: it's the exact step-by-step process you'll go through over the next 90 days, including the 3 places first-time buyers in this tier lose money and the Orlando-specific gotchas no out-of-state YouTube video will warn you about.
>
> [ Send me the 9-Step Roadmap → ]
>
> *Free. Delivered to your inbox in 60 seconds.*
>
> *After you've gone through the roadmap, if you want me to walk it with you, there's a free 30-minute Buyer Strategy Session linked at the end.*

**Tier B CTA target:** `/orlando-homebuying-roadmap/get?n={n}&src=fthb_lm1_tier_b`, where `{n}` is the URL-encoded first name from the result-page URL. **Do NOT include the email** in this URL — emails in shareable URLs leak PII; the opt-in form re-asks anyway. (LM2 task 03's opt-in form reads `?e=` and pre-fills if present, so the flow stays forward-compatible — just don't put it in the result-page URL now.) Fire `fthb_readiness_cta_click` with `{ tier: 'B', cta_target: 'fthb_lm2' }` on `pointerdown`.

---

## Tier C: Foundation Phase (score 0–44)

### Header

> # You're in the "Foundation Phase" tier.
> ### Your score: {{display_score}} / 100
> Based on your answers, the most useful timeline for you is **6 to 12 months** of focused foundation work before you start looking at homes. That sounds long. It isn't, and the work itself is mostly stuff you control without spending a dollar.

### Tier explanation paragraph

> If you landed here, it almost always means one of three things: your credit isn't where it needs to be yet, your savings aren't deep enough to cover down payment plus closing costs plus a reserve, or your timeline is honest. You're exploring, not buying yet. None of these are problems. They're just facts about today. The trap in this tier is one of two reactions: either you give up on the idea of homeownership entirely, or you push for a house anyway and end up with a mortgage payment that wrecks your finances for 5 years. Neither is necessary. The next section is what to actually do.

### The 2 mistakes Tier C buyers make most often

> ## The 2 mistakes you're most likely about to make
>
> **Mistake #1: Taking out a "credit-builder" loan or new card to "improve your score."**
> Credit advice on TikTok is often wrong for buyers. Opening new accounts in the 6 months before applying for a mortgage *lowers* your score short-term because the new account drops your average account age. The right moves in this tier are smaller and less satisfying: pull your full credit report from annualcreditreport.com (free, no card required), dispute any reporting errors, pay every account on time for 6 straight months, and bring revolving card balances below 30% of their limit. That's it. That's the playbook.
>
> **Mistake #2: Saving for the down payment in a checking account.**
> Money sitting in a checking account earns nothing and gets spent. If you're 6–12 months out from a purchase, every dollar of your home fund should be in a high-yield savings account (currently around 4%+ in 2026 in most online banks). Over 12 months on a $15,000 fund, that's an extra $600+ for closing costs. For doing nothing. Bonus: a separate, named account ("Orlando Home Fund") makes it psychologically harder to dip into.

### CTA (Tier C)

> ## What's next
> No call. No pitch. Not yet.
>
> What you need right now is **the right information at the right cadence**, not a real estate agent breathing down your neck. I'm going to send you one short, useful email every other week, covering credit, savings, market data, and common foundation-phase mistakes. When you cross into the 90-Day Sprint tier, I'll let you know and we'll talk then. Not before.
>
> *You're already enrolled. First email comes in about 48 hours.*
>
> *If you'd rather not get the emails, the unsubscribe link is at the bottom of every one. No hard feelings.*

**Tier C CTA target:** no CTA-out. It's a status message confirming Pipedrive enrolled them in bi-weekly nurture. The Hormozi rule for this tier is hard: **never pitch the BSS to Tier C**. If `fthb_readiness_cta_click` fires (it shouldn't really have anything clickable that counts as a CTA), pass `{ tier: 'C', cta_target: 'none' }`.

---

## Boilerplate footer (`<ResultFooter />`, used on every tier)

> *This scorecard is built by [Agent Name], a licensed REALTOR® in Florida. It's free, no information is sold or shared, and no agent contact happens unless you book the Buyer Strategy Session yourself. Have a question? Reply to any of my emails; I read them all.*
>
> *[License #] | [Brokerage]*

Replace `[Agent Name]`, `[License #]`, `[Brokerage]` with the agent's real values when wiring this up — the agent will supply them. Keep them as a small `src/config/agent.ts` if there's risk of them changing.

---

## Implementation notes

- **Score badge** uses the value passed from the URL. Style it visibly (large, color-coded by tier: A = green, B = amber, C = blue — but never color-only; pair with the tier name + a small icon).
- **CTA click tracking.** Each CTA fires `fthb_readiness_cta_click` with `{ tier, cta_target: 'bss' | 'fthb_lm2' | 'none' }` on `pointerdown` or `click`. `pointerdown` is more reliable in mobile browsers because navigation can cancel `click` handlers.
- **Validate numbers in copy.** Tier B Mistake #2 includes specific numbers (`$340K new build at 5.49%`, `$310K resale at 6.99%`, `$1,830`, `$1,955`, `$125/month lower`). Copy them verbatim. **Don't regenerate the math.** If a number changes later, edit this task file first, then propagate to the component.
- **Tier A CTA link constant.** `BSS_BOOKING_URL` lives in `src/config/links.ts` so the next phase can swap the booking URL without touching every component. The Tier A `<a href="...">` reads from this constant.

## Things NOT to do

- Don't paraphrase any block-quoted copy above. It passed the Hormozi tests in earlier review.
- Don't pitch the BSS on Tier C. Hard rule.
- Don't put the Tier B email into the result-page URL. Emails in shareable URLs leak PII; the LM2 opt-in form re-asks anyway.
- Don't soften the "2 mistakes" copy. The diagnostic edge is the value proposition.
- Don't use the word "corridor" in non-locked copy. Don't say bare "Downtown" — always "Downtown Orlando."

## Definition of Done

- [ ] All three tiers render the locked copy verbatim (block-by-block diff against this file)
- [ ] Score badge shows the value from the URL on every tier
- [ ] Tier A CTA links to `BSS_BOOKING_URL` from `src/config/links.ts`
- [ ] Tier B CTA links to `/orlando-homebuying-roadmap/get?n={n}&src=fthb_lm1_tier_b` (URL-encoded name; no email)
- [ ] Tier C has no CTA-out; it shows the confirmation copy
- [ ] `fthb_readiness_cta_click` fires on Tier A and Tier B CTAs with the right `cta_target` prop
- [ ] `<ResultFooter />` (license #, brokerage, "reply to my emails" note) appears on every tier
- [ ] `?preview=A`, `?preview=B`, `?preview=C` show the real content with stub names/scores

## Verification

Take the quiz three times with answer sets that produce A, B, and C. Confirm:
- Tier A page → BSS booking link works
- Tier B page → LM2 opt-in URL has `n=` and `src=fthb_lm1_tier_b` set correctly, no `e=`
- Tier C page → no CTA-out; the nurture confirmation reads correctly
