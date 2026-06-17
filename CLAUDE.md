# Faith Poirier Hypnotherapy — paid-search landing page

Static HTML/CSS landing page for the Google Ads campaign `FH | Search | Hypnotherapy`
(account 148-609-6952, Electro Raven MCC). No build step, no framework.

- **Target domain:** `lp.faithhypno.com` (the campaign's ad final URLs point here)
- **Deploy:** Vercel, as a static site. From this folder:
  `vercel --prod` (or connect the repo; framework preset = "Other", no build command, output = this dir).
  Then point `lp.faithhypno.com` at the deployment in Vercel → Domains.

## DKI
`?ag=` swaps the hero headline/subhead + `<title>` with no flash (class set in `<head>` before paint; default shows with JS off).
- `?ag=provider` · `?ag=sleep` · `?ag=anxiety` · anything else → `default`.

## Conversion tracking
Booking happens on **Heal.me** (off-domain), so a true "booking confirmed" event
can't be tracked. `book_call_click` (CTA click) is the proxy — it's pushed to
`window.dataLayer` on every CTA click. Reconcile against the Heal.me dashboard.
GTM container **GTM-PDGQPF7R** is installed (head + noscript) on both pages.
In GTM, add a Custom Event trigger on `book_call_click` and wire it to the
Google Ads conversion / GA4 event.

## Booking + images — DONE
- CTAs point to her free-consultation booking page: `https://faithhypno.com/services/free-15-minute-consultation-call` (her own domain; booking is Heal.me-backed)
- All 4 images are Faith's own, pulled from faithhypno.com (filestack CDN):
  hero = `R3Mk…` (virtual session), fit = `zaKj…` (headshot), session = `M1NI…`
  (in-person session), closer = `sS5I…` (garden-view session).

## BEFORE FLIPPING THE CAMPAIGN TO ENABLED
1. Deploy + point `lp.faithhypno.com` at the deployment (see Deploy above).
2. (Optional) Set the real `GTM-XXXXXXX` container id to capture `book_call_click`.
3. Verify on-page claims: "1,200+ Hours of Training" and the reviews (3 shown;
   hero says "Verified 5-star reviews" — add a real count if you want a number).

## Files
- `index.html` — the landing page (DKI hero, fit, session, reviews, closer, footer)
- `privacy.html` — privacy policy (linked in footer; needed for Google Ads)
