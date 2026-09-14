# Analytics & SEO — Record of What Was Set Up

**Last updated:** 2026-09-11
**Purpose:** A plain-English record of the analytics + search setup for morristownimprov.com, so that Liam (or an AI assistant like Claude) can get fully up to speed later without re-deriving anything. If you're an AI: read this file top to bottom and you'll know the current state, where everything lives, and how to make common changes.

> Companion file: `ANALYTICS-SEO-SETUP.md` is the *original step-by-step instructions*. **This file** is the *record of what actually got done* and how to navigate/change it going forward.

---

## 1. The one-paragraph summary

The site now has Google Analytics 4 (GA4) tracking visitors, wired in through Google Tag Manager (GTM), including custom tracking that reports section-to-section navigation (because the site is a single-page app). Google Search Console is verified and the sitemap is submitted, so Google will index the homepage. Everything is owned by Liam's personal Google account; his partner has Editor access to GA4. Nothing is secret in this file — the GA4/GTM IDs are already public in the site's HTML.

---

## 2. Where everything lives (the accounts map)

| Thing | Provider | Notes |
|---|---|---|
| **Domain registration** | **Spaceship.com** | Where the domain was bought / where it renews. **DNS is NOT managed here.** |
| **DNS + hosting/CDN** | **Cloudflare** | Nameservers: `maria.ns.cloudflare.com`, `elias.ns.cloudflare.com`. **All DNS changes happen in the Cloudflare dashboard**, not Spaceship. The site is deployed via a Cloudflare Git connection. |
| **Analytics** | **Google Analytics 4** | Under Liam's personal Google account. |
| **Tag management** | **Google Tag Manager** | Under Liam's personal Google account. |
| **Search / indexing** | **Google Search Console** | Under Liam's personal Google account. |
| **Code / source** | **GitHub** | Repo: `lwood-dev/morristownimprov`. Site deploys from the `main` branch. |

**Key mental model:** Spaceship = the store you bought the domain from. Cloudflare = where the domain actually "lives" and serves DNS + the website. Google = analytics and search, layered on top.

---

## 3. The IDs and identifiers

| Name | Value | Where it's used |
|---|---|---|
| GA4 Measurement ID | `G-R76FQ3950H` | Entered inside GTM (not directly in the site HTML). |
| GTM Container ID | `GTM-59ZTZFRZ` | Hardcoded in `index.html` (head script + body `<noscript>`). This is what loads all tracking. |
| GA4 property name | `Morristown Improv` | |
| GTM container name | `www.morristownimprov.com` | |
| Canonical site URL | `https://www.morristownimprov.com` | www is canonical; used in sitemap.xml, llms.txt, GA4 data stream. |

---

## 4. Exactly what was configured

### In the code (`index.html`, committed to GitHub)
- **GTM snippet** — a `<script>` in `<head>` and a `<noscript>` iframe right after `<body>`, both using container ID `GTM-59ZTZFRZ`. (These started as `GTM-XXXXXXX` placeholders and were swapped for the real ID.)
- **Virtual pageview tracking** — the site's section-navigation function pushes a `virtual_page_view` event to `dataLayer` on every section change, carrying `page_path` (e.g. `/#events`) and `page_title`. This is what lets analytics see section navigation on a single-page app.
- **CTA mailto click tracking** (added 2026-09-11) — every "contact us" style button on the site (email signup, interest survey, volunteer form, hire-us request, sponsor inquiries, etc.) works by building a `mailto:` link and handing it to the browser — it opens the visitor's own mail app with a pre-filled draft, but nothing is actually sent until *they* hit Send there. That's a real gap: a click is not a received email. All of these buttons funnel through one shared `mailTo(subject, bodyLines)` JS function (`index.html`, ~line 1716), so a single `dataLayer.push` there covers every CTA: event `cta_mailto_click`, with `cta_subject` (the exact string used as the email's subject line — matches what shows up in Liam's inbox) and `page_path`. See section 9 for how to use this to measure how much mailto "fallout" is actually happening.
- Also at the site root: `sitemap.xml` (lists the homepage for Google) and `llms.txt` (describes the org for AI crawlers).

### In Google Analytics 4
- Property **Morristown Improv**, US Eastern time, USD.
- One **Web data stream** for `https://www.morristownimprov.com`, which generated Measurement ID `G-R76FQ3950H`.
- **Users:** Liam (owner) + partner added as **Editor** (can view + edit, cannot manage users).

### In Google Tag Manager (container `GTM-59ZTZFRZ`)
Three objects, all **published**:
1. **Tag `GA4 - Config`** — type *Google Tag / GA4 Configuration*, Measurement ID `G-R76FQ3950H`, fires on **Initialization - All Pages**. (The base pageview on load.)
2. **Trigger `Virtual Pageview - Hash Nav`** — type *Custom Event*, event name `virtual_page_view`. (Listens for section changes.)
3. **Tag `GA4 - Virtual Pageview`** — type *GA4 Event*, uses the config above, event name `page_view`, with two parameters mapped to **Data Layer Variables**: `page_path` → `{{page_path}}` and `page_title` → `{{page_title}}`. Fires on the `Virtual Pageview - Hash Nav` trigger.

> **Why GA4 goes through GTM and not a direct gtag.js snippet:** so the virtual pageviews above can be tracked. If you ever paste the raw gtag.js "Google tag" snippet into the site *in addition* to GTM, it will **double-count** every visitor. Don't.

**⬜ Not yet wired up — needs Liam, same pattern as the virtual-pageview trigger above:** the code now pushes `cta_mailto_click` to `dataLayer` (see section 4), but nothing in GTM listens for it yet. To make it show up in GA4:
1. **Triggers** → **New** → **Custom Event** → event name `cta_mailto_click` → name it `CTA Mailto Click`.
2. **Variables** → **New** → **Data Layer Variable** → name `cta_subject`, Data Layer Variable Name `cta_subject` (same pattern as the existing `page_path`/`page_title` variables — GTM may already offer `page_path` as a built-in from the earlier setup).
3. **Tags** → **New** → **GA4 Event** → Configuration Tag `GA4 - Config` → Event Name `cta_click` → Event Parameters: `cta_subject` → `{{cta_subject}}`, `page_path` → `{{page_path}}` → Trigger `CTA Mailto Click` → name it `GA4 - CTA Mailto Click`.
4. **Submit** → **Publish**.
Once published, GA4 → **Reports → Engagement → Events** → `cta_click` will show a count; add `cta_subject` as a secondary dimension (or build an Exploration) to break it down per-button.

### In Google Search Console
- **Domain property** for `morristownimprov.com` (the Domain type covers www + non-www + http/https all at once).
- **Verified** via a **DNS TXT record** (`google-site-verification=...`) added in **Cloudflare** → DNS → Records.
- **Sitemap submitted** as the full URL `https://www.morristownimprov.com/sitemap.xml`. (Note: for a Domain property, the bare `sitemap.xml` is rejected as "invalid path" — you must give the full URL.)

---

## 5. How to test it's working

**Analytics (works immediately):**
1. analytics.google.com → **Morristown Improv** → **Reports → Realtime**.
2. Open the live site in another tab, click through sections.
3. You should appear as an active user and see the page path change per section.
4. *If nothing shows:* an ad-blocker is blocking your own visit — disable it or use another browser.

**Search (takes days to ~2 weeks):**
- Search Console → **Pages** / **Sitemaps** will gradually populate as Google crawls. Nothing to do but wait.

---

## 6. How to make common future changes

- **Add/remove someone's analytics access:** GA4 → **Admin (gear, bottom-left)** → **Property access management** → `+`. Only an Administrator can do this; currently only Liam.
- **Change/rename anything in tracking (tags, triggers):** GTM → open the `www.morristownimprov.com` container → edit → **Submit/Publish** (changes are NOT live until you Publish).
- **Add a DNS record (e.g. new verification, redirect):** **Cloudflare** dashboard → your site → **DNS → Records**. Not Spaceship.
- **Update event listings on the site:** these are hardcoded in `index.html` — `JAM_NEXT` object (~line 1658), `STL_UPCOMING` array (~line 1729), and the Meet-Up block in the HTML. Past dates auto-hide, but new events need a manual edit + commit + push. Buttons link out to live Eventbrite/Meetup pages.
- **Deploy a code change:** commit to `main` and push; the Cloudflare Git connection redeploys.

---

## 7. Known loose ends / tradeoffs (none urgent)

1. **No www ↔ non-www redirect.** Both versions currently load independently (both return 200). Minor SEO tidy-up: add a Cloudflare redirect rule sending non-www → www. Canonical is www.
2. **Everything is on Liam's personal Google account.** Single point of failure — only Liam can grant access. Fix later by making the partner an Administrator, or migrating GA4/GTM/Search Console to a dedicated Morristown Improv Google account.
3. **Single-page-app SEO limit.** The whole site is one URL; sections live behind `#` fragments that Google can't index separately. Only the homepage will ever rank in search. Fixing this means rebuilding as real routed pages — an architectural project, not a settings change.
4. **Events are manually maintained**, not pulled live from Eventbrite/Meetup. New events each month need a code edit. Optional future project: live Eventbrite API integration.

---

## 8. If you're an AI reading this to help Liam

- The IDs and structure above are current as of the "last updated" date. **Verify before assuming** — check `index.html` for the live GTM ID, and confirm anything account-side with Liam since you can't see his Google dashboards.
- Liam is non-technical; explain in plain language and give click-by-click steps for dashboard tasks. He prefers to run his own git commits/pushes.
- Google-account and Cloudflare tasks must be done by Liam in a browser (his logins). Code edits (like swapping IDs, updating events) can be done directly in the repo.

---

## 9. Sizing the mailto "fallout" — CTA clicks vs. emails actually received

**The concern:** every contact-style button on the site (signup, volunteer, hire-us, sponsor inquiry, etc.) is a `mailto:` link, not a real form submission. Clicking it just opens the visitor's own mail app with a draft pre-filled — nothing is sent until *they* hit Send over there. Some real, unknown fraction of clicks never turn into a message: the visitor is on a phone with no mail account configured, their browser blocks the mail-app handoff, they get cold feet at the draft screen, etc. Before spending effort replacing mailto with a real backend (e.g. a Cloudflare Worker form endpoint), it's worth finding out whether that drop-off is actually large enough to matter.

**The method:** compare two counts over the *same date range*:

1. **Clicks (intent)** — from GA4: **Reports → Engagement → Events** → filter to `cta_click` → add `cta_subject` as a secondary dimension (or build an **Exploration** with `cta_subject` as the breakdown) to see click volume per CTA. Requires the GTM trigger/tag in section 4 to be published first — until then this event is captured in `dataLayer` but not visible in GA4.
2. **Received (actual)** — in the `morristownimprov@gmail.com` inbox, search by the exact subject line for the same date range, e.g. `subject:"Email list signup" after:2026/9/1 before:2026/10/1`, and count results. Every CTA sets a fixed, distinct subject (table below), so this is a clean 1:1 match — no guessing which email came from which button.

| CTA (button text, page) | Exact email subject to search |
|---|---|
| Sign Up (Home, email list) | `Email list signup` |
| Submit (Get Involved, interest survey) | `Interest survey submission` |
| Request a Performance (Get Involved, community facility) | `Performance request for a community facility` |
| Volunteer Interest Form (Get Involved) | `Volunteer interest` |
| Ask About This (Sponsors, Monthly Headliner) | `Question about the Monthly Headliner add-on` |
| Request Our Partner Kit (Sponsors) | `Partner Kit request` |
| Start the Conversation (Sponsors) | `Interested in sponsoring Morristown Improv` |
| Request a Performance (Hire Us, private event) | `Private event booking request` |
| Request a Performance (Hire Us, Share the Laughs) | `Share the Laughs performance request` |

3. **Fallout rate** = `1 − (emails received ÷ clicks)`, per CTA and/or summed across all of them. A high number (say, over ~30-40%) is a real signal the mailto approach is costing real signups/bookings and worth prioritizing a fix; a low single-digit number means it's working fine as-is and other things are a better use of time.

**Caveats to keep in mind when reading the numbers:**
- GA4 undercounts clicks from visitors running ad-blockers or with "do not track"/consent settings that block GTM — so the click count is itself a floor, not exact.
- A visitor can also reply to a *previous* email with a new message that reuses/alters the subject line, or forward one to someone else who then emails in — minor noise, not worth correcting for at this volume.
- Give it a decent sample window (at least a few weeks, ideally a full month) before drawing conclusions — early on the numbers will be too small to mean anything.

**Status:** code-side click tracking is live (section 4); the GTM trigger/tag to surface it in GA4 is not yet published (section 4, "Not yet wired up"). Liam needs to do that step before clicks are visible in GA4 — the Gmail-subject-search side can be done any time, even retroactively for past months, since it's just search.
