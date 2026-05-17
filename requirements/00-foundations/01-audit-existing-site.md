
The funnel is being added to the existing **www.axelrivera.com** Astro site, not built from scratch. Before any LM1 or LM2 task starts, walk the existing repo and document its conventions. Output a single notes file that downstream tasks will reference instead of re-discovering project structure every session.

## Prereqs

None. This is task one.

## Context

The funnel is two static lead magnets that POST to a Make.com webhook, which creates a Pipedrive Person/Lead; Pipedrive Campaigns sends transactional and nurture email. The Astro side owns: rendering pages, running scoring client-side, building result URLs from query params, firing analytics, and POSTing payloads. **No backend, no cookies, no `LocalStorage`/`SessionStorage`, no env vars.** The Make.com webhook URL is baked into the build.

What the funnel will need from the existing site:

- A `BaseLayout` to wrap every funnel page (LM1 landing, quiz, result; LM2 landing, opt-in, rendered roadmap).
- An analytics provider (Plausible / PostHog / GA4 / Fathom) with a client-side `track()`-style helper for custom events.
- Tailwind (or a documented styling approach the funnel pages can adapt to).
- A vanilla-JS-friendly project structure (no framework islands for the quiz).
- Free routes under `/orlando-homebuying-readiness-quiz/*` and `/orlando-homebuying-roadmap/*`.
- A `public/assets/` (or equivalent) directory for a static PDF download.

## Goal

Produce **`requirements/EXISTING-SITE-NOTES.md`** in this repo. It is a stable reference document: every downstream task that asks about `BaseLayout`, analytics, Tailwind, or routing will defer to this file.

The notes file has a fixed section structure (below). Each section has a specific answer or a "**not present — needs to be added**" flag with a pointer to which downstream task should add it.

## Where to look

The existing site lives in a separate repo (locally checked out at a path the user can supply). For this task, **work in that repo** (read-only — you're auditing, not changing it). The output file goes back in this repo at `requirements/EXISTING-SITE-NOTES.md`.

If the existing repo isn't checked out locally yet, ask the user for its path before starting.

## Sections to fill out in EXISTING-SITE-NOTES.md

Use this template. Every section gets either a concrete answer or a "**not present — needs to be added in [task]**" line.

### 1. Project basics

- Repo location (absolute path on the developer's machine)
- Astro version (from `package.json`)
- Node version (from `.nvmrc` or `engines`)
- Build / deploy: hosting provider, deploy trigger, preview URL pattern
- Path aliases (e.g., is `~/` mapped to `src/`?) — from `tsconfig.json`
- TypeScript: strict mode? Any `// @ts-nocheck` smell?

### 2. Folder layout

A short tree of `src/` showing what's there:
```
src/
  pages/
  layouts/
  components/
  lib/         (or wherever shared TS modules live)
  scripts/     (or wherever client-side JS lives)
  styles/
  data/
  ...
```

Flag if any convention is non-standard. Pin where new code should land (`src/lib/webhook.ts` is the natural spot for the Make.com client — but only if the existing site uses `src/lib/`).

### 3. BaseLayout

- File path (likely `src/layouts/BaseLayout.astro` but confirm)
- Props signature — what does it accept? (`title`, `description`, `canonicalUrl`, anything else?)
- What does it render? (head meta, `<main>`, header, footer, skip-link?)
- Does it accept a `<slot>` for the body? Named slots for head injection?
- **Sample usage** — paste one block of a real existing page that uses `BaseLayout` so downstream tasks can copy the pattern verbatim

### 4. Analytics

- Provider (Plausible? PostHog? GA4? Fathom? something else?)
- Where is the script tag injected? (Almost certainly in `BaseLayout` — confirm)
- Is there an existing client-side helper? (e.g., `src/lib/analytics.ts` with a `track()` function)
- Function signature for firing custom events — **copy the actual signature** so downstream tasks can match it
- Auto page-view: does the provider fire one automatically on every page, or does the codebase fire it manually?
- Any existing custom events the funnel events should namespace around (to avoid collisions)?

### 5. Tailwind

- Present? (yes / no)
- If yes: config file path, any extended brand colors, custom fonts, custom plugins, custom `theme.extend`
- Brand color palette (primary, secondary, accent, neutral grays) — list the actual color tokens
- Typography defaults — what `@layer base` styles are set on `body` / headings?
- Container widths, breakpoints, any prose plugin in use
- If no Tailwind: what styling approach is used (vanilla CSS modules? CSS-in-JS? something else?), and decide whether the funnel pages add Tailwind or adapt to the existing approach. **Bias toward adapting to the existing approach** — adding Tailwind to a non-Tailwind codebase is its own multi-day project.

### 6. Brand and voice elements

This is the soft stuff that makes the funnel pages feel like part of the site:

- Heading font, body font, monospace font
- Primary color used for CTAs
- Button styling pattern — show one example button class string or component
- Form input styling pattern — show one example
- Iconography — any icon library in use (Heroicons, Lucide, Tabler, custom SVGs)?
- Existing site voice — is `BaseLayout` already opinionated about copy register (formal vs. conversational)?

### 7. Routing

- Are the planned funnel routes free?
  - `/orlando-homebuying-readiness-quiz` (LM1 landing)
  - `/orlando-homebuying-readiness-quiz/start` (LM1 quiz)
  - `/orlando-homebuying-readiness-quiz/result` (LM1 result)
  - `/orlando-homebuying-roadmap` (LM2 landing)
  - `/orlando-homebuying-roadmap/get` (LM2 opt-in)
  - `/orlando-homebuying-roadmap/view` (LM2 rendered roadmap)
  - `/assets/orlando-9-step-roadmap.pdf` (LM2 PDF)
- Confirm no existing page or asset already lives at any of those paths.
- Is `public/assets/` the right place for the PDF, or does the site use a different convention (e.g., `public/files/`)?

### 8. Forms and external integrations

- Does the site already POST to any external service (Formspree, Netlify Forms, a custom backend)? If so, document the pattern — we might be able to reuse it.
- Any existing CORS or CSP headers in `astro.config.mjs` or hosting config that could block a POST to a Make.com webhook URL?

### 9. Anything else worth flagging

Open-ended. Anything that surprised you during the audit — unusual build steps, a non-obvious convention, a hidden helper module, a partial migration in progress. Future you will thank present you for writing it down.

## Implementation notes

- **Be specific.** "Uses Tailwind" is useless. "Uses Tailwind 3.4 with brand colors `primary` (#1a73e8) and `accent` (#ff6b35) defined in `tailwind.config.cjs` at `theme.extend.colors`" is what downstream tasks need.
- **Copy real code.** When you find a `BaseLayout` usage pattern, paste an actual block from the existing site, not a paraphrase. Same for analytics calls, button styles, form inputs.
- **Flag gaps, don't fix them.** This task is read-only. If `BaseLayout` doesn't have a `description` prop and the funnel needs SEO descriptions, the notes say "BaseLayout currently doesn't accept `description`; [task to add it]." A separate task does the fix.
- **Don't reorganize the existing site to match the funnel's preferences.** The funnel adapts; the site doesn't.

## Things NOT to do

- Don't modify any file in the axelrivera.com repo during this task. Read-only.
- Don't write code in this repo other than `requirements/EXISTING-SITE-NOTES.md`.
- Don't assume Tailwind or strict TypeScript are present. Verify each.
- Don't paraphrase. Code patterns are easier to follow when quoted verbatim.

## Definition of Done

- [ ] `requirements/EXISTING-SITE-NOTES.md` exists in this repo
- [ ] Every section above has either a concrete answer or a "not present — needs to be added in [task]" line
- [ ] The BaseLayout, analytics helper, and button/input styling sections include **at least one verbatim code snippet** from the existing site
- [ ] Every planned funnel route is confirmed available (or a conflict is flagged with a proposed resolution)
- [ ] If Tailwind is missing, that's flagged with a recommendation (either: add Tailwind in this task; or: adapt the funnel pages to the existing styling approach — the audit picks one and explains why)

## Verification

- [ ] Open `requirements/EXISTING-SITE-NOTES.md` in a separate editor pane next to any LM1 or LM2 task file. Can you build the LM1 landing page from those two files alone, without re-opening the existing repo? If yes, the audit is good. If no, the audit is missing detail.
