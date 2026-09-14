# MARV Travel – website source (marvtravel.com)

One-page website running on the inPage CMS. The CMS theme (Bootstrap 3, "theme13") is
hidden with CSS and the visible page is built from the snippets in this repo.

## Files and where they go in inPage

| File | inPage location | Notes |
| --- | --- | --- |
| `hero.html` | Homepage → **motive box** (`#box-custom-motive`) | Icon font + preconnects + favicon swap, topbar, hero, trust strip. |
| `home.html` | Homepage → **page content** (`<main>`) | All sections, contact form and the page script. |
| `footer.html` | **Footer box** (`#box-custom-footer`) | Footer. |
| `style.css` | **Style 3** (`/style/3/`) | The only stylesheet we own. Loads after the theme. |
| `privacy-policy.html` | New page **Privacy policy**, slug `privacy-policy` | Linked from the form note and the footer. |

Deploy = copy-paste each file into its inPage field. The four homepage files must be deployed
together (the HTML uses classes that only exist in the new CSS). inPage's `custom_header` is
left alone (it is used for SEO); everything that would normally sit in `<head>` is at the top
of `hero.html`.

## What the page script does (`home.html`, bottom)

No external libraries any more (PureCounter and the Booking.com affiliate script were removed).

1. Topbar gets a shadow after scrolling.
2. Footer year is set automatically.
3. Counters animate when scrolled into view (`data-count` / `data-suffix`).
4. Contact form:
   * native validation with visible labels,
   * **honeypot** field `contact_time` (off-screen; bots fill it, humans can't),
   * **time gate**: a submit faster than 4 s after page load gets a "click once more" notice
     (humans click again, scripts don't),
   * **reCAPTCHA v3** loaded lazily when the form scrolls into view or gets focus, token
     obtained with action `submit`,
   * POST to the Make webhook via `fetch`; the form `action` no longer contains the webhook
     URL, so scrapers that harvest form actions get nothing,
   * clear success / error states, e-mail fallback when reCAPTCHA is blocked.

Fields sent to Make: `name`, `email`, `phone`, `address`, `message`, `contact_time`
(honeypot), `form_time_ms`, `source`, `g-recaptcha-response`.

## Make scenario "MARV Travel | Kontaktní formulář" – filter (applied 14 Sep 2026)

The scenario calls `siteverify`; the filter before the e-mail module now checks all of the
following (AND). Reply-To is set to the sender, and the e-mail shows the reCAPTCHA score.

| Field | Operator | Value |
| --- | --- | --- |
| `4.data.success` | boolean equal | `true` |
| `4.data.score` | number ≥ | `0.5` |
| `4.data.action` | text equal | `submit` |
| `4.data.hostname` | text equal | `www.marvtravel.com` |
| `1.contact_time` | does not exist (empty) | |
| `length(1.message)` | number ≥ | `20` |

Optional: a second route for "rejected" submissions that stores them in a Make data store, so
you can review false positives for a few weeks.

## Spam: where it really comes from

The Make scenario has had only about 20 runs since February 2025, so the webhook form is not
the spam channel. The inPage page **`/contact-form/`** has a native inPage form whose
"verification code" is printed in the HTML (`captcha[_captcha]` hidden field and
`/captcha/<code>` image), so any bot solves it. It is linked from the (hidden) inPage menu and
listed in `sitemap.xml`.

**Action:** unpublish or delete the `/contact-form/` page in inPage admin (or at least remove
the form and redirect the page to `/#contact`). Also remove `/search/`, `/site-map/` and
`/photo-galleries/` from the sitemap if inPage allows it – they are empty theme pages.

## Local preview

The audit harness used during development lives outside the repo (Playwright rendering the
live page with the local snippets swapped in). To reproduce: fetch `https://www.marvtravel.com/`,
replace the three boxes and `/style/3/` with the local files, open in a browser.
