
A reusable component (used on all 3 tier result pages) that renders the Orlando-metro price table plus the "three things this table doesn't tell you" callout. Data lives in a JSON file the agent updates monthly.


## Goal

A `<MarketSnapshot>` Astro component that takes no props and renders the full snapshot section from the locked copy below, reading data from `src/data/market-snapshot.json`. Used on all 3 tier result pages without modification.

## Files to create

```
src/components/MarketSnapshot.astro
src/data/market-snapshot.json
src/data/market-snapshot.schema.ts   (TypeScript type for the JSON, so updates fail to compile if they break shape)
```

## JSON shape

```ts
type MarketSnapshot = {
  anchorPrice: number;          // e.g., 500000
  snapshotMonth: string;        // e.g., "May 2026"
  areas: Array<{
    name: string;               // e.g., "Sanford"
    county: 'Seminole' | 'Orange';
    medianPriceFthb: number;    // e.g., 325000
    avgDaysOnMarket: number;    // e.g., 42
    anchorDescription: string;  // 1-sentence "what $500K gets you here"
  }>;
};
```

## Areas to include (price-band order: below-anchor → at-anchor → stretch)

In this order in the JSON:

1. Altamonte Springs (Seminole) — *well below anchor; mature suburb, room to negotiate at $500K*
2. Apopka (Orange) — *below anchor; north Orange, new-construction story, growing fast*
3. Sanford (Seminole) — *below anchor; biggest bang for the budget*
4. Winter Springs (Seminole) — *below anchor; school-driven demand, alternative to Oviedo*
5. Longwood (Seminole) — *at anchor; established, quiet, solid value*
6. Lake Mary (Seminole) — *at anchor; polished suburban option, strong resale*
7. Oviedo (Seminole) — *at anchor; perennial top search, school-driven*
8. Maitland (Orange) — *at anchor / slightly above; Winter Park's more affordable neighbor*
9. Winter Garden (Orange) — *above anchor; west-side outlier with luxury-skew median, FTHB-range inventory exists in the historic core*
10. Lake Nona (Orange) — *above anchor; entry-tier at $500K, full picture at $600K*

**Initial JSON values are placeholders the agent will replace with real MLS data before launch.** Use realistic-looking 2026 Orlando numbers (median price ~$370K–$560K across the set, DOM ~40–75 days). Default the `anchorPrice` to `500000` and `snapshotMonth` to the current month name.

## Locked copy (ship verbatim, rendered inline in the component — not in JSON)

The component renders this structure. Block-quoted text is final copy; `{{...}}` is data interpolation.

> ## What ${anchorPrice} actually buys across the Orlando metro right now
>
> *Updated {{snapshotMonth}}. Pulled from current MLS data, not Zillow's automated estimates. Anchored in the northern half of the metro — Seminole County and north Orange County — with two deliberate outliers (Winter Garden on the west, Lake Nona in the southeast) so you can see the full picture of what your budget unlocks.*
>
> | Area | County | Median price (FTHB range) | Avg. days on market | What ${anchorPrice} gets you |
> |---|---|---|---|---|
> | **Altamonte Springs** | Seminole | ${{alt_median}} | {{alt_dom}} | {{alt_anchor_description}} |
> | **Apopka** | Orange | ${{apk_median}} | {{apk_dom}} | {{apk_anchor_description}} |
> | **Sanford** | Seminole | ${{san_median}} | {{san_dom}} | {{san_anchor_description}} |
> | **Winter Springs** | Seminole | ${{ws_median}} | {{ws_dom}} | {{ws_anchor_description}} |
> | **Longwood** | Seminole | ${{lwd_median}} | {{lwd_dom}} | {{lwd_anchor_description}} |
> | **Lake Mary** | Seminole | ${{lm_median}} | {{lm_dom}} | {{lm_anchor_description}} |
> | **Oviedo** | Seminole | ${{ovd_median}} | {{ovd_dom}} | {{ovd_anchor_description}} |
> | **Maitland** | Orange | ${{mtd_median}} | {{mtd_dom}} | {{mtd_anchor_description}} |
> | **Winter Garden** | Orange | ${{wg_median}} | {{wg_dom}} | {{wg_anchor_description}} |
> | **Lake Nona** | Orange | ${{ln_median}} | {{ln_dom}} | {{ln_anchor_description}} |
>
> ### Three things this table doesn't tell you (and I will)
>
> 1. **Seminole County still has the strongest school-zoning resale floor** in the metro. Even older homes in Winter Springs and Longwood hold value because of zone demand. Orange-side homes (Maitland, Lake Nona) hold on **walkability, proximity to Downtown Orlando, and newer construction** instead — different drivers, similar resilience.
> 2. **Sanford, Altamonte Springs, and Apopka are the value plays** — $500K buys the most home in this report up there, with the trade-off being a 35–45 minute commute to Downtown Orlando. **Lake Nona is the stretch zone** in the southeast — at $500K you're at the entry tier (townhomes and smaller detached homes); $575–600K opens up Laureate Park and the kind of new construction that's hard to find in older Seminole neighborhoods. **Winter Garden is the west-side outlier** — strong downtown of its own, growing fast, often overlooked by buyers who only know the I-4 spine. The median is luxury-skewed, but historic-core inventory still exists near the anchor.
> 3. **Anything listed under $350K right now is almost always an HOA condo, a townhome, a manufactured home, or a single-family home with a non-obvious issue.** None of those are inherently wrong, but none are a typical first-home SFH purchase. Always ask what's underneath the headline price before getting attached.

## Implementation notes

- **Anchor price** is interpolated from `anchorPrice` in the JSON, so changing the anchor in the JSON updates every result page that embeds this component.
- **Format numbers** with `Intl.NumberFormat('en-US', { style: 'currency', currency: 'USD', maximumFractionDigits: 0 })`. So `500000 → "$500,000"`.
- **Table is responsive.** On phone widths (375px), switch to a stacked card layout (one card per area). On desktop, the standard table. Tailwind's container queries are overkill; a `md:` breakpoint is enough (or adapt to the host site's styling approach per `EXISTING-SITE-NOTES.md`).
- The "three things this table doesn't tell you" callout has fixed copy — render it inline in the component, not in the JSON.
- **Property-type priority:** bias example listings the agent picks for `anchorDescription` toward **single-family homes first, townhomes second, condos only when SFH/townhome inventory is genuinely thin** at the anchor price in that area. The snapshot exists to set expectations for SFH/townhome buyers, who are the funnel's primary audience.

## Snapshot data update process (operational note for the agent)

The market snapshot table is dynamic data but updates infrequently. The agent updates it **once per calendar month** by:

1. Pulling current median list price + average DOM from MLS for each named area (Seminole side: Altamonte Springs, Sanford, Winter Springs, Longwood, Lake Mary, Oviedo; Orange side: Apopka, Maitland, Winter Garden, Lake Nona).
2. Browsing 3 active listings in each area within $25K of the `anchorPrice` to write the "what you get" description. For Lake Nona specifically, the median sits above the anchor — bias the example listings toward the entry tier ($475–510K) so the "what $500K gets you" copy stays honest, and include one $600K reference listing for the "stretch unlocks Laureate Park" framing. Bias toward SFH first, townhomes second.
3. Updating the values in `src/data/market-snapshot.json`.
4. Updating `snapshotMonth` to the new month name.

A reminder for this monthly update should be set as a scheduled task once the funnel ships.

## Things NOT to do

- Don't pull data from a remote API. The snapshot updates monthly; a JSON commit is the right cadence.
- Don't bake the data into the component's source. Updates need to be a single-line JSON edit, not a code edit.
- Don't change the area set without the agent's say-so. The 10 names above are the canonical buyer-facing list; College Park, Audubon Park, Winter Park (standalone), Eatonville, Casselberry, Belle Isle, Conway, Dr. Phillips, Kissimmee, Clermont, Horizon West, Baldwin Park, Thornton Park, and St. Cloud are out of scope unless the agent explicitly adds one. College Park and Audubon Park were considered but dropped because the median sits ~$75–100K above the $500K anchor and entry-tier inventory is too thin to anchor the report there.
- Don't reintroduce the old "I-4 corridor between Sanford and Downtown Orlando" framing in any headline or callout copy. The snapshot is now a metro-wide picture anchored in the north with two outliers, not a corridor view.
- Don't include condos as the default "what $X gets you" examples; bias toward single-family homes first, townhomes second.
- Always say "Downtown Orlando," never bare "Downtown" — the metro has many downtowns.

## Definition of Done

- [ ] `<MarketSnapshot />` renders the locked table + callout above
- [ ] Renders on a phone (375px) without horizontal scroll
- [ ] Editing `src/data/market-snapshot.json` updates the rendered page on `npm run dev` reload
- [ ] TypeScript prevents the JSON from drifting from the schema (e.g., adding `county: 'Volusia'` errors at build)
- [ ] All 10 areas appear in the order above (Altamonte Springs → Lake Nona, price-band order)
- [ ] `anchorPrice` in the JSON drives every "$X buys" mention in the rendered output

## Verification

Edit `anchorPrice` to `500000` in the JSON. Reload. Every dollar mention in the headline + table column header should update. Revert.
