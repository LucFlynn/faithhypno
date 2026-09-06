# Faith Poirier Hypnotherapy — paid-search landing page

Static HTML/CSS landing page for the Google Ads campaign `FH | Search | Hypnotherapy`
(account 148-609-6952, Electro Raven MCC). No build step, no framework.

- **Live domain:** `book.faithhypno.com` (the campaign's ad final URLs point here;
  it currently serves `index.html`, the v1 page)
- **Deploy:** Vercel, as a static site. From this folder:
  `vercel --prod` (or connect the repo; framework preset = "Other", no build command, output = this dir).
  Preview/staging deploy: `vercel` (needs `vercel login` first — the CLI token in
  this environment is expired).

## Two pages, one campaign
- `index.html` — **v1, currently live.** MECLABS-sequenced page built around the
  free consult.
- `index-v3.html` — **v3, staging.** Written from scratch against the "Landing
  Page Copy" and "Direct-Response Warm-Up Piece" docs: video-forward hero, two VSL
  slots, and a PDF lead magnet with optional email capture. Not live yet.
- `index-v4.html` — an earlier draft of the same page, kept for comparison.

## DKI (both pages)
`?ag=` swaps the hero headline/subhead + `<title>` with no flash (class set in
`<head>` before paint; the default angle still shows with JS off).
- `?ag=provider` · `?ag=sleep` · `?ag=anxiety` · anything else → `default`.
Ad final URLs already carry these, so any replacement page must keep them.

## v3 — what still needs real inputs
Both live in one CONFIG block at the bottom of `index-v3.html`:

1. **The two VSLs** — `VIDEOS.primary` / `VIDEOS.secondary` (`src` + `aspect`,
   plus `poster` for a self-hosted file). Empty = the styled placeholder stays up,
   so the page is safe to ship without them.
   - **both are live**: Cloudflare Stream embeds, both shot vertical (`aspect: "9/16"`).
     `primary` is the short hook in the hero; `secondary` is the longer one in the
     dark band. Swap the two `src` values to reverse that.
   - `aspect` must match the footage. A vertical primary also needs
     `class="hero hero--portrait"` on the hero `<section>`, which puts the tall
     player beside the copy on desktop instead of under it. Swap in a landscape
     video and that class comes off.
   - Self-hosted MP4/WebM renders in a native player (see `assets/video/README.md`);
     Cloudflare Stream, YouTube, Vimeo and Wistia URLs become embeds.
   - `vsl_play` / `vsl_progress` come from the `<video>` element for self-hosted
     files and from Cloudflare's player SDK (loaded on demand) for Stream embeds.
     A YouTube or Vimeo embed would report neither without its own listener.
2. **`OPTIN_ENDPOINT`** — where the guide opt-in posts (any endpoint taking a
   JSON POST: Formspree, Kit, Mailchimp, a Vercel function). Left empty, the guide
   stays a plain download link and no contact details are collected. The markup
   ships as that download link and JS upgrades it into the form, so the guide is
   reachable with JS off either way.

## The lead magnet
- `assets/switch-off-sequence-guide.html` — source of the 6-page guide.
- `assets/switch-off-sequence-guide.pdf` — what visitors download. Regenerate after
  editing the source:
  `"/Applications/Google Chrome.app/Contents/MacOS/Google Chrome" --headless --no-pdf-header-footer --print-to-pdf=assets/switch-off-sequence-guide.pdf file:///Users/luc/faithhypno/assets/switch-off-sequence-guide.html`
- `assets/switch-off-sequence-cover.png` — the cover thumbnail shown on the page;
  it is a screenshot of page 1 of the guide, so re-shoot it whenever the cover changes.

## Funnel (v3) — guide first, call second
The booking-first version of this page produced no bookings, so v3 leads with the
lead magnet and treats the call as step two:

1. **Hero → a stacked CTA pair** (`.cta-stack`): the guide (`.guide-jump`, an
   on-page jump to `#guide`, fires `guide_cta_click`) and the call, both visible,
   same width. Order and weight are driven off the `?ag=` class in CSS — the top
   one is solid, the second is a ghost outline under a connector line
   ("Ready to talk now?" / "Not ready to talk yet?"). `provider` leads with the
   call; every other angle leads with the guide. Mid-page CTA stays guide-only.
2. **The opt-in** captures name + email, fires `guide_lead_submit`, delivers the PDF.
3. **The hand-off** — the panel that replaces the form is a booking ask
   (`book_call_click` with `placement: post-guide`), because someone who just took
   the guide is the warmest reader on the page.
4. **Booking stays reachable throughout** for high-intent visitors: the header
   button, the hero's paired call button (`hero-secondary`), the `#free-call`
   section, and the closer.
5. **The sticky dock swaps offer by scroll depth** — guide until the pricing
   tiers come into view, then the call (`.dock.is-call`), because past the price
   the reader has what they need to decide. `?ag=provider` gets the call the
   whole way down, in CSS, with no JS involved.

Two consequences worth holding onto:
- **`OPTIN_ENDPOINT` is now load-bearing.** Empty, the primary CTA hands out a PDF
  and captures nothing at all. The page logs a console warning in that state.
- **In Google Ads, `guide_lead_submit` should become the primary conversion** while
  booking volume is zero — Smart Bidding cannot optimize toward a conversion that
  never fires. Keep `book_call_click` as a secondary/observation conversion and
  move it back to primary once bookings have real volume.

To revert to booking-first, swap the `.guide-jump` CTAs in the markup back to
`.book` links pointing at the consultation URL.

## Conversion tracking
Booking happens on **Heal.me** (off-domain), so a true "booking confirmed" event
can't be tracked. `book_call_click` (CTA click) is the proxy. GTM container
**GTM-PDGQPF7R** is installed (head + noscript) on every page. Every event carries
`ad_group` (from `?ag=`).

| dataLayer event | fired when | extra params |
| --- | --- | --- |
| `guide_cta_click` | a "Get the Free Guide" CTA (v3) | `placement` (hero/video-two/dock) |
| `book_call_click` | any booking CTA | `placement` (masthead/hero-secondary/free-call/closer/post-guide) |
| `guide_lead_submit` | guide opt-in submitted (v3) | `asset` |
| `guide_lead_error` | opt-in POST failed; PDF served anyway (v3) | `asset` |
| `guide_download_click` | "Download the Guide" re-click (v3) | `asset` |
| `vsl_loaded` | a video slot mounts (v3) | `video_slot` |
| `vsl_play` | first play, self-hosted or Cloudflare Stream (v3) | `video_slot` |
| `vsl_progress` | 25/50/75/100% watched, self-hosted or Stream (v3) | `video_slot`, `percent` |
| `faq_open` | an FAQ item is opened (v3) | `question` |

In GTM, `guide_lead_submit` is the conversion to optimize on for now, with
`book_call_click` tracked alongside it (see Funnel above). `vsl_progress` fires for
self-hosted files and Cloudflare Stream — a YouTube/Vimeo embed would need its own
GTM listener.

## Booking + images — DONE
- CTAs point to her free-consultation booking page: `https://faithhypno.com/services/free-15-minute-consultation-call` (her own domain; booking is Heal.me-backed)
- All images are Faith's own, pulled from faithhypno.com (filestack CDN):
  hero = `R3Mk…` (virtual session), fit = `zaKj…` (headshot), session = `M1NI…`
  (in-person session), closer = `sS5I…` (garden-view session).

## BEFORE PUTTING v3 IN FRONT OF PAID TRAFFIC
1. Both VSLs are in. Check `vsl_play` / `vsl_progress` land in GTM preview once
   deployed - the Stream SDK wiring could not be exercised locally.
2. Set `OPTIN_ENDPOINT` — **blocking**. The primary CTA is the opt-in now, so
   without it the page's main action captures nothing. Needs a follow-up email
   sequence behind it too, or the addresses just sit there.
3. Deploy to a preview URL for Faith to review, then promote to `book.faithhypno.com`.
4. Verify on-page claims: "1,200+ Hours of Training", the three reviews, and the
   $225 / $195 pricing.

## Files
- `index.html` — v1 landing page (live)
- `index-v3.html` — v3 landing page (staging)
- `index-v4.html` — earlier draft of the same page
- `privacy.html` — privacy policy (linked in footer; needed for Google Ads)
- `assets/` — lead magnet source, PDF, cover image, and `video/` for the VSLs
- `tmp/` — local render/screenshot scratch, not part of the deploy
