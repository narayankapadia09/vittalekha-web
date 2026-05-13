# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Static marketing site for **VittaLekha**, an AI recordkeeping app for small businesses. Three pages: [index.html](index.html) (landing), [privacy.html](privacy.html), [terms.html](terms.html). Deployed to Vercel.

The product is branded as "AI recordkeeping" in user-facing copy. The keywords meta and JSON-LD on [index.html](index.html) intentionally also include "bookkeeping" so the site still ranks for that search term — leave those alone unless explicitly asked.

## Commands

There is no build step, package manager, or test suite. Workflows:

- **Local preview**: open `index.html` directly in a browser, or serve the directory (e.g. `python -m http.server 8000`). Tailwind is loaded from CDN, so no compile step.
- **Deploy**: `vercel` (or `vercel --prod`). The `.vercel/` directory is gitignored and holds the project link.

## Architecture

### Tailwind config is duplicated per page
Tailwind is loaded via the `cdn.tailwindcss.com` CDN, and the custom theme (brand colors, fonts, shadows) is defined inline in a `<script>tailwind.config = {...}</script>` block inside each HTML `<head>`. **When changing theme tokens (colors like `brand`, `cream`, `ink`; the `Inter` font; `boxShadow.soft`/`ring`), update all three HTML files** — there is no shared config file. The full token set lives in [index.html](index.html#L26-L43); privacy/terms carry a slimmer subset.

### Custom CSS lives in styles.css; Tailwind utilities everywhere else
[styles.css](styles.css) holds the things Tailwind alone cannot express: scroll-reveal transitions (`.reveal` / `.visible`), the phone mockup + floating cards (`.phone-frame`, `.float-card-{1,2,3}`), keyframe animations (`phoneFloat`, `cardFloat`, `scanBeam`, `bubbleIn`, `extractIn`, `progress`, `blink`), and the legal-page `.legal-section` styling. Class names referenced from HTML must match what is defined here.

### script.js wires two behaviors
[script.js](script.js) is plain vanilla JS, no modules:
1. An `IntersectionObserver` adds `.visible` to any `.reveal` element when it scrolls into view (one-shot — `unobserve` after firing). Removing the `reveal` class from an element makes it appear immediately with no animation.
2. Click handler on `a[href^="#"]` for smooth-scroll to in-page anchors.

### Vercel headers
[vercel.json](vercel.json) sets `cleanUrls: true` (so links to `/privacy` resolve to `privacy.html`) and pins long-lived `Cache-Control: immutable` on `styles.css` and `script.js`. **Bust the cache by renaming the file or appending a query string in the HTML `<link>`/`<script>` tags** when you ship CSS/JS changes — otherwise returning visitors will see the old asset for up to a year. HTML files are not pinned and update normally.

### Cross-page consistency
The header nav, footer, and favicon/OG meta are copy-pasted across `index.html`, `privacy.html`, `terms.html`. Brand changes (logo path, copyright year, contact email `hello@vittalekha.com`, links) must be applied to all three.

### SEO assets
[robots.txt](robots.txt) and [sitemap.xml](sitemap.xml) live at the site root and reference `https://vittalekha.com`. Update the canonical domain in **all** of these places at once if it ever changes: `robots.txt`, `sitemap.xml`, the `<link rel="canonical">`, `og:url`, and the JSON-LD `url` fields in [index.html](index.html). The JSON-LD block in [index.html](index.html#L52) describes the app as a `SoftwareApplication` with pricing, rating, and feature list — keep these aligned with the visible pricing/feature copy.
