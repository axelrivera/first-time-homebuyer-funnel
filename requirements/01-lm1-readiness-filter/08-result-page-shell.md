
The static result page that reads `?n=...&t=A|B|C&s=...` from the URL, validates, and renders the matching tier's section. Tier-specific copy lands in the next task; this one builds the routing + fallback + preview mode.

## Prereqs

- [01-07 Market snapshot component](./07-market-snapshot-component.md) (embedded inside the result page on every tier).

## Goal

`/orlando-homebuying-readiness-quiz/result.astro` reads three query params, validates them, and renders one of three tier branches (A, B, C) or a fallback. The page is fully static and works without client-side JS (render-time logic lives in the `.astro` frontmatter). `?preview=A|B|C` overrides the URL params with stub data so the agent can review.

## Files to create

```
src/pages/orlando-homebuying-readiness-quiz/result.astro
```

## Why no HMAC / signed token / expiry

The data is the user's own self-reported answers. Pipedrive holds the authoritative record (from the Make.com webhook). A user can tamper with their URL to display a different tier; **nothing they can do with that affects the agent's CRM, the email sequence they're enrolled in, or which CTA gets the most warm lead**. A tampered Tier-A page link that gets booked into the BSS gets qualified in conversation; no harm done. The page works forever from a valid `?n=…&t=…&s=…` URL.

The result page **never** trusts the URL for anything beyond rendering. No conditional that affects state (e.g., "this person is Tier A, send them this thing") may key off the query params.

## Query params

| Param | Meaning | Allowed values |
|---|---|---|
| `n` | First name (URL-encoded) | Any string; `decodeURIComponent`-ed, trimmed, HTML-escaped on render. Empty/missing → render as "you" in copy. |
| `t` | Tier letter | `A`, `B`, `C` (case-insensitive). Anything else → fallback view. |
| `s` | Display score (0–100) | Integer 0–100. Anything else → render the tier without a score badge (still useful), or fall back if the tier is also bad. |

## Result page structure (all three tiers; copy filled in next task)

1. **Header** with tier name + score badge (e.g., "Maria, you're in the 90-Day Sprint tier." + `{displayScore} / 100`).
2. **Tier explanation paragraph** (next task).
3. **The 2 mistakes you're most likely about to make** (tier-specific; next task).
4. **`<MarketSnapshot />`** — same on every tier.
5. **The "What's Next" CTA** (tier-specific; next task).
6. **Email confirmation line**: *"We just emailed you a copy of this page. Check your spam if you don't see it in 2 minutes."*
7. **`<ResultFooter />`** (boilerplate footer; next task).

## Implementation notes

### Read params in the `.astro` frontmatter

```ts
const url = new URL(Astro.request.url);
const previewTier = url.searchParams.get('preview'); // 'A' | 'B' | 'C' | null
const nParam = url.searchParams.get('n');
const tParam = url.searchParams.get('t');           // 'A' | 'B' | 'C' | null
const sParam = url.searchParams.get('s');
```

### Validation

- Tier letter must be `'A'`, `'B'`, or `'C'` (case-insensitive). Anything else → fallback view.
- Score must parse as an integer in `[0, 100]`. Anything else → render the tier without a score badge (still useful), or fall back if the tier is also bad.
- Name (`n`) is decoded with `decodeURIComponent` and HTML-escaped on render. Empty/missing → fall back to "you" in the headline copy.

### Preview mode (`?preview=A|B|C`)

- Use stub data: `firstName = 'Maria'`, score = `92` for A, `68` for B, `34` for C.
- Ignore the other params if preview is set.
- Render a small "PREVIEW MODE" ribbon (top of page, `bg-yellow-100` or equivalent) so the agent doesn't confuse it with a real result.

### Fallback view (malformed params, no preview)

- Headline: "Your result has been emailed to you."
- Body: "If you don't see it in 2 minutes, check spam. Or take the scorecard again." with a link back to `/orlando-homebuying-readiness-quiz`.
- This task scaffolds the fallback; copy is final.

### Branch by tier in the page body

For each tier, render `<TierAContent />`, `<TierBContent />`, or `<TierCContent />` — these components are placeholders for this task. The next task fills them with real copy. For now, they can be stubs returning `<div>Tier A content here</div>`.

### Other

- **Embed `<MarketSnapshot />`** between the (placeholder) mistakes section and the (placeholder) CTA on every tier.
- **No JS required to render.** All conditional logic is in the `.astro` frontmatter; the rendered HTML is final. The only client JS is the analytics event + CTA click tracking (next task).
- **Fire `fthb_readiness_result_view`** on page load with `{ tier, score }` props. Skip in preview mode.

## Things NOT to do

- Don't decode the params in client JS — render-time decoding makes the page work with JS disabled.
- Don't compute scoring on this page. The score comes in via query param. The scoring engine ran once on the quiz page.
- Don't validate the URL against an HMAC, signed token, or anything similar. See the rationale above.
- Don't add a "share my result" button. Sharing is the user's choice; the URL is already shareable.

## Definition of Done

- [ ] `/orlando-homebuying-readiness-quiz/result?n=Maria&t=A&s=92` renders the Tier A branch with name "Maria" and score 92
- [ ] `?n=Maria&t=B&s=68` renders Tier B
- [ ] `?n=Maria&t=C&s=34` renders Tier C
- [ ] `?n=Maria&t=Z&s=99` renders the fallback view
- [ ] `?preview=A` renders Tier A with stub data and the PREVIEW ribbon
- [ ] The market snapshot appears on all 3 tier views
- [ ] Page renders correctly with JavaScript disabled in the browser (DevTools → Settings → Disable JavaScript, then reload)
- [ ] `fthb_readiness_result_view` fires with `tier` and `score` props on real (non-preview) loads
- [ ] Lighthouse a11y score ≥ 95 on the rendered tier pages

## Verification

```
http://localhost:4321/orlando-homebuying-readiness-quiz/result?n=Maria&t=A&s=92
http://localhost:4321/orlando-homebuying-readiness-quiz/result?n=Maria&t=B&s=68
http://localhost:4321/orlando-homebuying-readiness-quiz/result?n=Maria&t=C&s=34
http://localhost:4321/orlando-homebuying-readiness-quiz/result?n=&t=X&s=abc   # fallback
http://localhost:4321/orlando-homebuying-readiness-quiz/result?preview=B      # preview
```
