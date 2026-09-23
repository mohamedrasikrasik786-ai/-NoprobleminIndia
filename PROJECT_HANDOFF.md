# Project handoff — No Problem in India landing page

This file is a one-time orientation doc for a new AI/tool picking up this project cold (written because the client is switching from Claude Code to Google Antigravity on a duplicated copy of this directory). Read this first, then `CLAUDE.md` for the full technical decision-log.

## What this project is

A single-page marketing/journal site for **No Problem in India** — a Polish–Indian family (Renata & Sridhar) running a permaculture farm and field journal in rural Tamil Nadu. The client runs the real business (farm visits, a WhatsApp soap shop, a YouTube/Instagram/blog presence) and this page is their web front door: hero, story, "how we live" pillars, farm & products, gallery, testimonials, journal/blog carousel, a community CTA, and a footer.

## The one file that matters

**`landing-page.html`** is the entire deliverable — a single self-contained HTML file (~4.45MB), no build step, no framework, no external dependencies except Google Fonts. Every photo/image on the page is embedded directly as a `data:image/...;base64,...` URI inside the HTML. This is deliberate, not an oversight: the page is developed and hosted as a Claude Artifact, and the Artifact sandbox cannot fetch external image URLs, so every image had to be inlined.

**Practical consequence for editing tools**: several lines in this file (one per embedded photo) are several hundred KB to a few MB of base64 text on a *single line*. Standard "read whole file" or "read N lines" tools will choke on these — don't try to read or diff the file naively. When you need to edit HTML near an image line without touching the image itself:
- Use a plain-text search (grep/ripgrep) for short surrounding markers (class names, `alt` text, comments) rather than reading the line itself.
- For structural HTML edits near these lines (moving a block, wrapping a div), write a tiny script (Node, Python, whatever your tool has) that does `readFile` → `indexOf`/string-replace on short unique anchor strings → `writeFile`, rather than trying to load-and-diff the whole document. This project used small throwaway Node scripts for exactly this several times.
- CSS and JS edits (in the `<style>` and `<script>` blocks near the top/bottom of the file) are totally normal small text and can be read/edited directly — it's only the six-ish image-carrying HTML lines in the body that are oversized.

Two other files in this directory (`index.html`, `styles.css`, `image 19.png`) are **unrelated leftovers**, not part of this build — don't deploy them, don't treat them as sources of truth. `assets/` holds a few source images (logo badge, background photo) that got base64-encoded into `landing-page.html`.

## Where it's published

- **Claude Artifact** (dev/preview copy, updates freely on every change): a private Artifact URL the client can open in a browser. Not usable by a different AI tool unless it also has Artifact publishing — otherwise just work off the local file and preview it in a browser/dev server instead.
- **Vercel production** (`no-problem-in-india` project): `https://no-problem-in-india.vercel.app` — the real live site. **Only redeploy here when the client explicitly says "deploy."** Every other change stays local until that word is said. See `CLAUDE.md`'s "Deploying (current, actual process)" section for the exact commands — the short version: MCP-style one-shot deploy tools can't handle a 4MB+ file, so it's deployed via the local Vercel CLI from a clean scratch folder (`index.html` at the folder root, not inside `src/`).

## Design system (the site's actual rules — don't reinvent these)

All defined as CSS custom properties in `landing-page.html`'s `:root`:

```css
--color-primary: #5D936F;       /* CTA buttons, accents */
--color-primary-hover: #4A7D5C;
--color-accent: #8DBC90;        /* secondary buttons/links, header logo */
--color-dark: #264933;          /* headings, dark brand sections */
--color-bg-base: #FEFBF8;       /* light section background (alternates with bg-alt) */
--color-bg-alt: #ECE2D6;
--color-bg-warm: #F2D2AC;
--color-brown: #503E28;         /* small accents only, light backgrounds only */
--color-text-primary: #264933;
--color-text-body: #3A3A3A;
--color-text-muted: #7A7A7A;
--color-yellow: #F6CE40;        /* hero section only, plus its blob mask */
--gutter: 64px desktop / 40px ≤1024px / 16px ≤640px;
--radius: 16px desktop / 12px ≤760px;   /* applies to buttons + cards, NOT the two hero photo-frame accents */
```

Fonts: `Fraunces` for most headings, `Plus Jakarta Sans` (800 weight) for the hero + "Join the community"/footer headings specifically (a Cal Sans substitute — Cal Sans isn't on Google Fonts), `Work Sans` for body text, `Caveat` for handwritten-style accents.

Corners were flattened to square sitewide, then explicitly reversed to `var(--radius)` on client request — see CLAUDE.md's "Rounded corners are BACK" section for exactly which elements are in/out of scope (buttons + cards yes; the scrollbar thumb, the "Our mission"-style tag/badge, and the two Figma hero photo-frame accents stay square).

## Things that will bite you if you don't know them

1. **The file must keep a real `<!DOCTYPE html><html><head><meta name="viewport" content="width=device-width, initial-scale=1">...` structure.** It was missing entirely for a long stretch of this project's history, and browsers silently "fixed" it visually on desktop while real phones defaulted to a ~980px virtual viewport and never fired any `max-width` mobile CSS at all. If mobile styles ever stop working sitewide (not just one section), check for this tag before anything else.
2. **Real product data is live, not placeholder**: the Farm & Products section lists 5 real WhatsApp-catalog soap SKUs and prices (Creamy Care ₹325, The Blackout ₹205, Golden Glow ₹205, Acne Cleanse ₹205, Buzzing Beauty ₹205 — each "was" a higher price, shown as a sale). Verify with the client before changing these; a WhatsApp catalog can change without notice and this was accurate as of 2026-09-20.
3. **The client does not want em-dashes anywhere** in copy or in your own written responses to them — use periods/commas/colons instead. This is a stated communication preference, not a design rule, but it'll come up if you're asked to write or edit any on-page copy.
4. **Never redeploy to Vercel without an explicit "deploy"** from the client, no matter how small the change. Artifact/local updates are fine constantly; the live site is not.
5. **A `.gallery.html` link already exists in the markup** (the "View Full Gallery" button in the Gallery section) but the page itself doesn't exist yet — it's a deliberately placeholder dead link, built ahead of the real page per client request.
6. This project is **not a git repository**. There's no version history beyond what's in the Claude Artifact's own version list and whatever the client has manually saved. Be careful with destructive file operations — there's no `git stash` safety net here.

## For the deep history

`CLAUDE.md` in this same directory is a long, chronological, very detailed decision-log covering the hero section's Figma-fidelity build, the Community & Footer redesign (background image math, mobile photo-bleed mechanism, footer card layout), several real bugs found and fixed (missing viewport tag, a `min-height:100%` CSS quirk that silently doesn't resolve against a parent height, a mobile parallax that "felt broken" but was actually fine — the backdrop behind it was just flat black), and the exact reasoning behind dozens of specific pixel/breakpoint choices. It's written for an AI editing this same file, so read it before touching the hero, the footer, or anything mobile-responsive — several of those areas have already had multiple failed approaches tried and reverted, and the notes explain why so you don't repeat them.
