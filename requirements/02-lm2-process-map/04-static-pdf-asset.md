
Drop the agent-produced PDF at the path the LM2 transactional email links to. Mostly an operational task — the Astro side just needs the file to exist at the right URL.

## Prereqs

- The agent has designed the PDF in Canva (or similar) and exported it. The content is the same as the rendered roadmap at `/view`, just laid out for print.

## Goal

`/assets/orlando-9-step-roadmap.pdf` is reachable at the public URL after `npm run dev` and `npm run build`. The LM2 transactional email's PDF link resolves to this URL in production.

## Files to create

```
public/assets/orlando-9-step-roadmap.pdf
```

(That's it. One file.)

## Why a download link, not an email attachment

Pipedrive Campaigns is the sender, not Make.com. Linking to a stable URL is one line of merge-tag-free HTML. Static hosting on the Astro site is free (it's part of the existing deploy), the file is CDN-fast for the reader, and the link can be re-shared (the agent will hear "can you send me that PDF again?" — a URL is the answer). The transactional email and the inline `/view` page share the same source of truth: whatever's in `/public/assets/` at the most recent deploy.

If PDF *generation* (from the live web view) ever becomes worth it, that's a Phase 4+ optimization. Not now.

## Implementation notes

- **Astro convention:** anything in `public/` is served at the corresponding root URL. `public/assets/foo.pdf` → `https://<domain>/assets/foo.pdf`.
- **The PDF itself** is produced by the agent in Canva or similar. This task is the wiring: drop the file at the right path, verify the URL works, update Pipedrive Campaigns' merge field to point at the production URL.
- **Size budget.** A roadmap PDF should be 1–3 MB, not 20 MB. If the Canva export is huge, run it through a free PDF compressor (Smallpdf, ILovePDF) before committing. Large PDFs hurt email deliverability — some inboxes will warn the reader.
- **File naming.** Match exactly: `orlando-9-step-roadmap.pdf`. The transactional email merge field is wired to this filename.
- **Don't version the filename.** Each redeploy replaces the file in place. If the agent wants the previous version preserved, that's git history.
- **PDF path checks `EXISTING-SITE-NOTES.md` Section 7** — if the existing site uses a different convention (e.g., `public/files/` instead of `public/assets/`), match the host convention and update the URL accordingly. Then the transactional email's merge field gets the new URL.

## Pipedrive Campaigns merge field

The LM2 transactional email links to this PDF via a merge field. After deploying the PDF, update Pipedrive Campaigns so the `{{fthb_roadmap_pdf_link}}` (or whatever the field is named) resolves to the production URL. That's an operational step done in Pipedrive's UI, not in the codebase, but it's part of this task's DoD.

## Things NOT to do

- Don't generate the PDF from the `/view` page at build time. The spec deliberately ships a hand-designed PDF for the initial release.
- Don't put the PDF in `src/assets/`. That path is for files Astro processes; we want the PDF served raw.
- Don't commit a PDF over 5 MB without checking with the agent. If the file is that large, it's probably high-resolution image bloat that won't print better.

## Definition of Done

- [ ] `public/assets/orlando-9-step-roadmap.pdf` (or host-convention equivalent path) is committed and under 5 MB
- [ ] `curl -I http://localhost:4321/assets/orlando-9-step-roadmap.pdf` returns 200 with `Content-Type: application/pdf` in dev
- [ ] `npm run build` includes the file in `dist/assets/`
- [ ] After deploy, `https://<production-domain>/assets/orlando-9-step-roadmap.pdf` returns 200
- [ ] Pipedrive Campaigns' PDF-link merge field is set to the production URL
- [ ] Opening the PDF in a browser shows the full 9-step content (not a placeholder)

## Verification

```bash
npm run dev
curl -I http://localhost:4321/assets/orlando-9-step-roadmap.pdf
# HTTP/1.1 200 OK
# Content-Type: application/pdf
```

Then trigger an LM2 opt-in end-to-end and confirm the email Pipedrive sends contains a working link to the PDF.
