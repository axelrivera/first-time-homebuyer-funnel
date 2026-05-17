
A reusable component (used on all 3 tier result pages) that renders the I-4 corridor price table plus the "three things this table doesn't tell you" callout. Data lives in a JSON file the agent updates monthly.


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
  anchorPrice: number;          // e.g., 475000
  snapshotMonth: string;        // e.g., "May 2026"
  areas: Array<{
    name: string;               // e.g., "Sanford"
    county: 'Seminole' | 'Orange';
    medianPriceFthb: number;    // e.g., 325000
    avgDaysOnMarket: number;    // e.g., 42
    anchorDescription: string;  // 1-sentence "what $475K gets you here"
  }>;
};
```

## Areas to include (north to south, both sides of I-4 between Sanford and Downtown Orlando)

In this order in the JSON:

1. Sanford (Seminole)
2. Lake Mary (Seminole)
3. Altamonte Springs (Seminole)
4. Winter Springs (Seminole)
5. Maitland / Winter Park (Orange)
6. College Park (Orange)
7. Apopka — corridor side (Orange)

**Initial JSON values are placeholders the agent will replace with real MLS data before launch.** Use realistic-looking 2026 Orlando numbers (median price ~$310K–$450K range for FTHB inventory, DOM ~30–60 days). Default the `anchorPrice` to `475000` and `snapshotMonth` to the current month name.

## Locked copy (ship verbatim, rendered inline in the component — not in JSON)

The component renders this structure. Block-quoted text is final copy; `{{...}}` is data interpolation.

> ## What ${anchorPrice} actually buys along the I-4 corridor right now
>
> *Updated {{snapshotMonth}}. Pulled from current MLS data, not Zillow's automated estimates. All areas are along the I-4 corridor between Sanford and Downtown Orlando.*
>
> | Area | County | Median price (FTHB range) | Avg. days on market | What ${anchorPrice} gets you |
> |---|---|---|---|---|
> | **Sanford** | Seminole | ${{san_median}} | {{san_dom}} | {{san_anchor_description}} |
> | **Lake Mary** | Seminole | ${{lm_median}} | {{lm_dom}} | {{lm_anchor_description}} |
> | **Altamonte Springs** | Seminole | ${{alt_median}} | {{alt_dom}} | {{alt_anchor_description}} |
> | **Winter Springs** | Seminole | ${{ws_median}} | {{ws_dom}} | {{ws_anchor_description}} |
> | **Maitland / Winter Park** | Orange | ${{wp_median}} | {{wp_dom}} | {{wp_anchor_description}} |
> | **College Park** | Orange | ${{cp_median}} | {{cp_dom}} | {{cp_anchor_description}} |
> | **Apopka (corridor side)** | Orange | ${{apk_median}} | {{apk_dom}} | {{apk_anchor_description}} |
>
> ### Three things this table doesn't tell you (and I will)
>
> 1. **Seminole County still has the strongest school-zoning resale floor** in the metro. Even older homes in Winter Springs hold value because of zone demand. Orange-side homes (College Park, Winter Park) hold on **walkability and proximity to Downtown Orlando** instead — different driver, similar resilience.
> 2. **Sanford is the value play on the north end** of the corridor, but the trade-off is a 35–45 minute commute to Downtown Orlando. **Apopka's corridor side is the value play on the west**, with a shorter Downtown Orlando commute but fewer top-tier school zones.
> 3. **Anything listed under $325K right now is almost always an HOA condo, a townhome, a manufactured home, or a single-family home with a non-obvious issue.** None of those are inherently wrong, but none are a typical first-home SFH purchase. Always ask what's underneath the headline price before getting attached.

## Implementation notes

- **Anchor price** is interpolated from `anchorPrice` in the JSON, so changing the anchor in the JSON updates every result page that embeds this component.
- **Format numbers** with `Intl.NumberFormat('en-US', { style: 'currency', currency: 'USD', maximumFractionDigits: 0 })`. So `475000 → "$475,000"`.
- **Table is responsive.** On phone widths (375px), switch to a stacked card layout (one card per area). On desktop, the standard table. Tailwind's container queries are overkill; a `md:` breakpoint is enough (or adapt to the host site's styling approach per `EXISTING-SITE-NOTES.md`).
- The "three things this table doesn't tell you" callout has fixed copy — render it inline in the component, not in the JSON.
- **Property-type priority:** bias example listings the agent picks for `anchorDescription` toward **single-family homes first, townhomes second, condos only when SFH/townhome inventory is genuinely thin** at the anchor price in that area. The snapshot exists to set expectations for SFH/townhome buyers, who are the funnel's primary audience.

## Snapshot data update process (operational note for the agent)

The market snapshot table is dynamic data but updates infrequently. The agent updates it **once per calendar month** by:

1. Pulling current median list price + average DOM from MLS for each named area (Seminole side: Sanford, Lake Mary, Altamonte Springs, Winter Springs; Orange side: Maitland/Winter Park, College Park, corridor-side Apopka).
2. Browsing 3 active listings in each area within $20K of the `anchorPrice` to write the "what you get" description. Bias toward SFH first, townhomes second.
3. Updating the values in `src/data/market-snapshot.json`.
4. Updating `snapshotMonth` to the new month name.

A reminder for this monthly update should be set as a scheduled task once the funnel ships.

## Things NOT to do

- Don't pull data from a remote API. The snapshot updates monthly; a JSON commit is the right cadence.
- Don't bake the data into the component's source. Updates need to be a single-line JSON edit, not a code edit.
- Don't widen the area list to "all of Orlando metro" or narrow it to Seminole-only. Lake Nona, Dr. Phillips, Kissimmee, Clermont are out of scope unless the agent says otherwise.
- Don't include condos as the default "what $X gets you" examples; bias toward single-family homes first, townhomes second.
- Don't use the word "corridor" in copy beyond the locked headline phrase ("I-4 corridor between Sanford and Downtown Orlando"). And always say "Downtown Orlando," never bare "Downtown" — the metro has many downtowns.

## Definition of Done

- [ ] `<MarketSnapshot />` renders the locked table + callout above
- [ ] Renders on a phone (375px) without horizontal scroll
- [ ] Editing `src/data/market-snapshot.json` updates the rendered page on `npm run dev` reload
- [ ] TypeScript prevents the JSON from drifting from the schema (e.g., adding `county: 'Volusia'` errors at build)
- [ ] All 7 areas appear in the order above (Seminole-side north to south, then Orange-side)
- [ ] `anchorPrice` in the JSON drives every "$X buys" mention in the rendered output

## Verification

Edit `anchorPrice` to `500000` in the JSON. Reload. Every dollar mention in the headline + table column header should update. Revert.
