# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

Single-page static marketing website for F. Bonavise Trucking, a family-owned Long Island freight delivery company. There is **no build system, no dependencies, and no package manager** — the entire site is one hand-authored file, `index.html`.

## Running / Developing

- Preview: open `index.html` directly in a browser, or serve the folder (`python3 -m http.server`).
- No build, lint, or test step exists. Edits to `index.html` are the deliverable.
- The contact form's AJAX submit (`fetch('/', …)`) only works when served by Netlify (see below); it will fail locally, which is expected.

## Architecture

Everything lives in `index.html`:
- **CSS** is inline in a single `<style>` block in `<head>`. The design system is driven by CSS custom properties in `:root` (colors like `--navy`, `--cream`, `--signal`; fonts `--disp` Oswald / `--body` Inter). Reuse these variables rather than hardcoding values — changing a brand color should mean editing one `:root` entry.
- **JS** is a single inline `<script>` at the end of `<body>` (vanilla, no libraries). It handles three things: the mobile menu toggle (`#menutoggle` / `#navlinks`), scroll-reveal animations via `IntersectionObserver` on any element with class `.reveal`, and the contact form submit.
- **Page structure** is a sequence of `<section>` blocks with anchor ids used by the nav: `#services`, `#coverage`, `#about`, `#contact`. Fonts load from Google Fonts; the logo is embedded as a base64 `data:image/png`.

### Scroll reveal
Add the `.reveal` class to any element to have it fade/animate in on scroll — the `IntersectionObserver` picks it up automatically, no per-element wiring needed.

### Contact form (Netlify Forms)
The `#contactForm` is wired for **Netlify Forms** (`data-netlify="true"`, hidden `form-name` input, honeypot `bot-field`). The JS intercepts submit, POSTs form-encoded data to `/`, and on success swaps `#formFields` for the `#formSuccess` message; on failure it re-enables the button and shows a "call us" fallback. Keep the hidden `form-name` input and honeypot field intact — Netlify relies on them. The primary contact fallback throughout the site is the phone number **631-639-6744**.

## Assets

The `images/` directory holds source photos/logos (HEIC, JPEG, PNG). Two image patterns are in use:

- **Logo / hero art**: inlined as base64 `data:` URIs directly in `index.html` (one very long line, ~275). Updating these means replacing the data URI, not dropping a file in `images/`.
- **Gallery photos**: web-optimized JPEGs in `images/gallery/`, referenced by `<img loading="lazy">` (not inlined, to keep `index.html` from bloating). Source phone photos are large and some are HEIC (which browsers can't display) — convert/resize with `sips` before adding, e.g. `sips -s format jpeg -Z 1600 -s formatOptions 80 SRC.HEIC --out images/gallery/name.jpg`. Note some iPhone JPEGs carry an EXIF orientation flag, so the browser's decoded width/height can be transposed vs. what `sips` reports; set the `<img width/height>` from the browser's `naturalWidth`/`naturalHeight`.

The gallery is a CSS-grid mosaic (`.gal-grid`, one `.feature` tile spans 2×2) with a vanilla-JS lightbox (`#lightbox`) supporting click, prev/next, backdrop-click, Esc, and arrow keys. Each `.gal-item` is a `<button>` carrying a `data-cap` caption; the lightbox reads `src`/`alt`/`data-cap` off the items, so adding a photo needs no JS changes.
