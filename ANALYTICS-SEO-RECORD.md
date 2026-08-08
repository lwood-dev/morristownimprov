# Analytics & SEO — Record of What Was Set Up

**Last updated:** 2026-08-07
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
