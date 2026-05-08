# Marketing site patch — invite landing + Universal Links

Drop these files into `stuartridout/getswapd` (the GitHub Pages repo
that serves https://getswapd.com) to make invite links work end-to-
end.

## What this adds

- `index.html` — homepage at `/`. Hero, how-it-works, screenshot
  strip, why-swapd, footer. References `assets/swapd.css` and the
  web-sized images in `assets/screenshots/`.
- `assets/swapd.css` — shared stylesheet linked from every HTML
  page. Marmalade palette, Bricolage Grotesque + Caveat fonts (via
  Google Fonts), and a reusable `.swapd-wordmark` component.
- `assets/screenshots/` — web-sized JPEGs of the App Store
  screenshots used by the homepage strip. Regenerate by re-running
  `scripts/build-appstore-screenshots.py` (in the app repo) and the
  resize step at the top of that script's docstring.
- `.well-known/apple-app-site-association` — so iOS recognises
  `https://getswapd.com/join/<CODE>` as a Swapd Universal Link and
  opens the app directly when it's installed.
- `.well-known/assetlinks.json` — same for Android App Links.
  **Fill in the SHA-256 fingerprint** (see below) before shipping,
  otherwise Android falls back to the browser.
- `404.html` — catches `/join/<CODE>` paths (GitHub Pages has no
  per-code file, so it 404s); the inline script rewrites to
  `/join/?code=<CODE>` where the real landing page lives. Any
  other 404 renders a plain "not found" message.
- `join/index.html` — the invite landing page. Reads `?code=`,
  shows the code in big letters, offers a store-install button
  (which copies the code to the clipboard first), plus an "I
  already have Swapd" button that fires `swapd://join/<CODE>`.
- `support/index.html` — public help page at `/support`. Covers
  invites, accounts, notifications, common questions, and how to
  contact us.
- `privacy/index.html` — full privacy policy at `/privacy`. The
  app links to this from the onboarding rules screen, the
  re-accept screen, and the You tab. Content must stay in sync
  with the app's `PRIVACY_VERSION` constant — see "Version sync"
  below.
- `privacy/simple/index.html` — AADC plain-language version at
  `/privacy/simple`, aimed at under-16s. Same commitments as the
  full policy, short sentences, no jargon.
- `terms/index.html` — terms of use at `/terms`. Same sync ritual
  as privacy — version must match `TERMS_VERSION` in the app.
- `safety/index.html` — safety page at `/safety`. In-app reporting
  flow, Childline / IWF / NSPCC signposting, and the safety email
  address.
- `.nojekyll` — GitHub Pages hides dotfiles under Jekyll by
  default. This switches Jekyll off so `/.well-known/*` actually
  gets served.

## Version sync (important)

The privacy and terms pages have version strings baked into the
copy ("Last updated: 19 April 2026"). The app has matching
constants at `constants/LegalVersions.ts`:

```
export const TERMS_VERSION = '2026-04-19';
export const PRIVACY_VERSION = '2026-04-19';
```

When the server on a device sees a user whose accepted versions
don't match these constants, it hard-gates them into the
re-accept screen. So the ritual when you change either document:

1. Edit `privacy/index.html` or `terms/index.html` here in
   `marketing-patch/`, bump the "Last updated" date.
2. Copy the same file into `stuartridout/getswapd`.
3. Bump the corresponding constant in
   `constants/LegalVersions.ts` in the app repo to the new date.
4. Ship both together so users see the new copy at the same time
   they're re-prompted to accept.

## Before committing

One placeholder still needs filling in:

1. **Android signing fingerprint** in `.well-known/assetlinks.json`
   Replace the empty `sha256_cert_fingerprints` array with the
   SHA-256 from Play Console → App integrity → App signing key
   certificate. Android won't verify the App Link without it.

The App Store URL is already wired to the live listing
(`apps.apple.com/gb/app/swapd-fun-photo-swaps/id6762082136`). The
Android Play Store link in `join/index.html` currently points back
to `/support#android-beta` because Android is in closed testing —
swap it for `https://play.google.com/store/apps/details?id=app.swapd`
once the public listing goes live.

## How to apply

From the `getswapd` repo root:

```sh
# copy the patch directory into the repo root, preserving structure
cp -R /path/to/inwoven/marketing-patch/. .
git add .nojekyll .well-known 404.html join
git commit -m "Add invite landing page + Universal Links config"
git push
```

After GitHub Pages redeploys (usually under a minute), test:

```sh
curl -I https://getswapd.com/.well-known/apple-app-site-association
curl https://getswapd.com/.well-known/apple-app-site-association
curl -I https://getswapd.com/join/TESTCODE
# ^ expect a 404 status but with the landing page HTML
```

Apple's validator lives at
<https://app-site-association.cdn-apple.com/a/v1/getswapd.com>
— it refreshes every ~24 hours after the AASA file first goes up.

## Source of truth

The same JSON is currently committed in the Next.js server
(`server/src/app/.well-known/...`). Once this patch is live, delete
those server routes — they'll never be hit because getswapd.com
doesn't point at the Next.js server.
