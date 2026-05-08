# Handoff — swapd marketing site brand refresh

This document hands off the brand-refresh work in `marketing-patch/`
to a Claude session running in the **`stuartridout/getswapd`** repo.
The patch was prepared in `stuartridout/inwoven` (the app repo), where
Claude doesn't have permission to push to the marketing repo.

## What you're applying

A site-wide visual refresh from the old teal/cream palette to the
**Marmalade** palette used in the iOS app, plus a homepage rewrite
and the live App Store URL wired in.

### TL;DR for whoever is reviewing

1. Copy the seven HTML files plus the `assets/` directory from this
   patch into the repo root. Overwrite freely.
2. Decide what to do with the existing `index.html` (see Conflicts).
3. Verify the four routes below render before deploying.
4. Fill the one outstanding placeholder (Android SHA-256).

## Files in the patch

```
marketing-patch/
├── index.html                       ← NEW homepage (hero + how-it-works + strip + footer)
├── 404.html                         ← refreshed
├── assets/
│   ├── swapd.css                    ← NEW shared stylesheet (linked from every page)
│   └── screenshots/                 ← NEW web-sized JPEGs (~80–120 KB each)
│       ├── 01-today.jpg
│       ├── 02-lock-in.jpg
│       ├── 03-reveal.jpg
│       ├── 04-roll.jpg
│       └── 05-lane.jpg
├── join/index.html                  ← refreshed; App Store URL now wired up
├── support/index.html               ← refreshed; new "Android in testing" callout
├── safety/index.html                ← refreshed
├── privacy/index.html               ← refreshed
├── privacy/simple/index.html        ← refreshed
├── terms/index.html                 ← refreshed
├── README.md                        ← documents the patch (read this if you haven't)
└── .well-known/                     ← unchanged: AASA + assetlinks.json
```

Everything outside `assets/` already had a counterpart in this patch
in earlier versions, so the copy/overwrite step is mechanical.

## Brand decisions (so you don't have to re-derive them)

- **Palette:** Marmalade. Tokens are CSS custom properties at the top
  of `assets/swapd.css` (`--paper`, `--ink`, `--accent`, etc.). They
  mirror `constants/Theme.ts` in the app repo — keep them in sync.
- **Typography:** Bricolage Grotesque (display, 700–800), Caveat
  (hand-script, 500–700), Inter Tight (UI, 400–700). All loaded from
  Google Fonts via the `@import` at the top of `swapd.css`.
- **Wordmark:** lowercase `swapd` in Bricolage 800, with a
  hand-drawn terracotta wavy underline (inline SVG, mirrors
  `components/SwapdWordmark.tsx`). Reusable as `.swapd-wordmark`
  with `.large` / `.small` modifiers. **The brand is always
  lowercase**, even at the start of a sentence.
- **Voice:** lowercase `swapd` in body copy. The noun "swap" is also
  lowercase ("a swap with someone you know"). The only exception is
  iOS Settings paths like *Settings → Swapd → Notifications* which
  match the OS display name from `app.json`.
- **App Store CTA:** swapd-styled pill (paper background) with the
  Apple glyph SVG + "Download on the App Store" text, **not** Apple's
  official badge. This is intentional — it complies with Apple's
  guidelines and looks like part of the site rather than pasted in.
  If your team wants the official badge instead, swap the `<a>` for
  an `<img>` of `https://tools.applemediaservices.com/api/badges/...`.

## App Store + Android

| Surface | Link |
|---|---|
| iOS | `https://apps.apple.com/gb/app/swapd-fun-photo-swaps/id6762082136` (live) |
| Android (public listing) | `https://play.google.com/store/apps/details?id=app.swapd` (placeholder; not wired anywhere) |
| Android (current) | `/support#android-beta` — email `hello@getswapd.com` to join the closed beta |

Once Android ships publicly, change the Android CTA in three places:
1. `index.html` hero — both `<a class="btn btn-secondary" href="/support#android-beta">` blocks
2. `index.html` end-CTA — same selector
3. `join/index.html` — `var PLAY_STORE_URL = ...` near the top of the
   inline script

Search-and-replace `"/support#android-beta"` →
`"https://play.google.com/store/apps/details?id=app.swapd"` will catch them all.

## Conflicts to expect

1. **Existing `index.html`.** This patch now ships a new homepage.
   If the live `getswapd.com/` already has a homepage you want to
   keep parts of (hero copy, partner logos, anything not visible to
   me), diff before overwriting. The new file is self-contained and
   can be replaced wholesale, edited in place, or used as a starting
   point.
2. **Existing `/assets/` directory.** If `getswapd` already has an
   `assets/` folder, merge rather than overwrite — the new files
   only land at `assets/swapd.css` and `assets/screenshots/*.jpg`.
3. **Inline styles in old pages.** All seven HTML pages dropped
   their per-page `<style>` blocks and now `<link rel="stylesheet"
   href="/assets/swapd.css">`. If any pages in the live site
   weren't tracked in this patch (e.g. a `/press` or `/about` page),
   they'll still look old until you migrate them — point me at the
   file list and I can refresh those too.
4. **Old `.well-known/apple-app-site-association`.** Untouched in
   this refresh, but Apple's AASA must match the bundle ID and
   App Store ID. If your existing AASA was a placeholder, refresh
   from the `id6762082136` listing.

## Outstanding placeholders

- **`.well-known/assetlinks.json`** — empty `sha256_cert_fingerprints`
  array. Pull the SHA-256 from Play Console → App integrity → App
  signing key certificate and paste it in. Until then, Android falls
  back to the browser instead of opening the app for App Links.

## Verification checklist

After copying the files:

```bash
# from the getswapd repo root
python3 -m http.server 8080
```

Then open in a browser and visually check:

- [ ] `/` — hero renders with marker-highlight on "One"; both CTAs
  visible; screenshot strip shows 5 phones; layout collapses to
  single column at narrow widths.
- [ ] `/join/?code=ABC123` — code shows in big letters; "Download
  on the App Store" button copies the code to clipboard then opens
  the App Store; "I already have swapd" fires `swapd://join/ABC123`.
- [ ] `/support` — Android-in-testing callout appears at the top of
  the body copy.
- [ ] `/privacy`, `/privacy/simple`, `/terms`, `/safety` — wordmark
  in the header row, marmalade palette throughout, no leftover teal
  (`#0C6E72`, `#052E30`) anywhere.

Also check view-source for any of:
- `0C6E72`, `052E30`, `8A8A92`, `F2F8F8` (old teal tokens)
- `id0000000000` (old App Store placeholder)
- Capitalised `Swapd` outside iOS Settings paths

Anything that survives is a missed file or a regression.

## What I changed in `inwoven` while preparing this

For traceability, the matching commits on `main` in `inwoven` are:

- `1cf25cf` — Refresh marketing-patch to swapd Marmalade brand
- `24bbf58` — Add swapd marketing homepage at marketing-patch/index.html

Both merges already include the same set of files this handoff
references, so `marketing-patch/` in `inwoven@main` is the source of
truth for what to copy across.

## Questions for the receiving session

If you hit any of these, ping back to the inwoven session for an
answer rather than guessing:

- The live homepage has sections this draft doesn't (testimonials,
  press logos, an FAQ). Should they be reinstated?
- The voice notes in the screenshots show prompts that are also
  used in the in-app Founding Five. Are any of these on a
  do-not-publish list for marketing?
- Should the Apple App Store badge be the official Apple-supplied
  one rather than the swapd-styled CTA in this draft?
