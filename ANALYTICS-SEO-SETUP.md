# Analytics & SEO Setup — morristownimprov.com

*Mirrors the GA4 + GTM + Search Console + llms.txt setup used on karlring.com.*

**Note for Claude Code:** this repo already has `llms.txt`, `sitemap.xml`, and the GTM snippet/virtual-pageview tracking committed (see "What's already done" below) — pull latest before starting. Steps marked **Repo** below (editing `index.html`, committing, pushing) are yours to execute directly once Liam hands you a real GTM Container ID. Steps marked **Google account** need Liam himself in a browser — he owns those logins, they can't be done from the CLI. After Liam completes the Google-side steps and gives you the real IDs, swap the placeholder `GTM-XXXXXXX` in `index.html`, commit, and push.

---

## What's already done in the code (no action needed)

- **`llms.txt`** — added at the site root. A plain-language summary of the org for AI assistants/crawlers, following the emerging [llms.txt](https://llmstxt.org) convention.
- **`sitemap.xml`** — added at the site root, pointing Google at the homepage.
- **GTM snippet** — installed in `index.html` (head script + body `<noscript>` fallback), currently using a placeholder container ID (`GTM-XXXXXXX`) that needs to be swapped for the real one — see Step 2.
- **Virtual pageview tracking** — this site is a single-page app that changes sections via URL hash (`#events`, `#about`, etc.) without a real page reload. Out of the box, GA4 would only ever log **one pageview per visit**, no matter how many sections someone browsed — a real gap for a group that wants to know if people are actually checking the Events or Sponsors pages. The site's router now pushes a `virtual_page_view` event to `dataLayer` on every section change, with `page_path` (e.g. `/#events`) and `page_title`. GTM needs one extra trigger to use this — see Step 3.

---

## Step 1 — Create the GA4 property

1. Go to [analytics.google.com](https://analytics.google.com) and sign in with whichever Google account should own this (a dedicated org account is cleaner long-term than a personal one, same consideration as the Cloudflare account).
2. **Admin** → **Create Property** → name it "Morristown Improv" → time zone US Eastern, currency USD.
3. Choose **Web** as the platform, enter `https://www.morristownimprov.com` as the site URL.
4. This creates a **Measurement ID** (`G-XXXXXXXXXX`). Copy it — you'll need it in Step 3, not for the HTML directly (GA4 gets wired up *through* GTM, not installed separately).

## Step 2 — Create the GTM container

1. Go to [tagmanager.google.com](https://tagmanager.google.com) → **Create Account** → account name "Morristown Improv", container name `morristownimprov.com`, target platform **Web**.
2. This gives you a **Container ID** (`GTM-XXXXXXX`).
3. In the repo, open `index.html` and replace both occurrences of `GTM-XXXXXXX` (one in the `<head>` script, one in the `<body>` `<noscript>` iframe) with your real container ID.
4. Commit and push — this redeploys automatically once the Cloudflare Git connection is fixed (see the separate note on that), or upload manually via the usual zip process in the meantime.

## Step 3 — Wire GA4 into GTM (two tags, one trigger)

Inside the GTM container ([tagmanager.google.com](https://tagmanager.google.com)):

**Tag 1 — GA4 Configuration (base pageview on load):**
1. **Tags** → **New** → **GA4 Configuration**.
2. Measurement ID: paste the `G-XXXXXXXXXX` from Step 1.
3. Trigger: **Initialization - All Pages** (built-in).
4. Name it `GA4 - Config`.

**Trigger — Virtual Pageview (for section changes):**
1. **Triggers** → **New** → **Custom Event**.
2. Event name: `virtual_page_view`.
3. Name it `Virtual Pageview - Hash Nav`.

**Tag 2 — GA4 Event (fires on section change):**
1. **Tags** → **New** → **GA4 Event**.
2. Configuration Tag: select `GA4 - Config` from Tag 1.
3. Event Name: `page_view`.
4. Event Parameters: add `page_path` → Data Layer Variable `page_path`, and `page_title` → Data Layer Variable `page_title`. (Create these as new **Data Layer Variables** under Variables if GTM doesn't offer them as built-ins — Variable name matches the `dataLayer` key exactly, e.g. `page_path`.)
5. Trigger: `Virtual Pageview - Hash Nav`.
6. Name it `GA4 - Virtual Pageview`.

**Publish:** top-right **Submit** → give the version a name like "Initial GA4 setup" → **Publish**.

## Step 4 — Verify it's actually firing

1. In GTM, click **Preview**, enter `https://www.morristownimprov.com`. This opens the site with GTM's debug panel attached.
2. Click through a few nav links (Events, About, Sponsors) and confirm `virtual_page_view` events show up in the debug panel, and that `GA4 - Virtual Pageview` fires each time.
3. In GA4, go to **Reports** → **Realtime** and confirm pageviews show up as you click around — you should see the page path change per section, not just one static hit.

## Step 5 — Google Search Console

1. Go to [search.google.com/search-console](https://search.google.com/search-console) with the same Google account.
2. **Add Property** → **URL prefix** → `https://www.morristownimprov.com`.
3. Verify ownership. Since Liam already manages the domain's DNS in Cloudflare, the **DNS TXT record** method is the path of least resistance — Search Console gives you a TXT record to add; add it in Cloudflare DNS (Dashboard → DNS → Records → Add record → type TXT), no code changes or redeploy needed. (The alternative HTML meta-tag method would require another code push — skip it if DNS access is available.)
4. Once verified, go to **Sitemaps** in the left nav → submit `sitemap.xml` (i.e. enter `sitemap.xml`, Search Console appends the domain).
5. Google won't index instantly — expect it to take anywhere from a few days to a couple weeks for the homepage to show up in search results. Nothing else to do here; Search Console will start reporting coverage and any crawl issues.

---

## A known limitation worth knowing about

Because this is a hash-routed single-page app, Google only ever sees **one real URL** — `https://www.morristownimprov.com/` — everything after `#` is invisible to search engines regardless of the sitemap or Search Console setup. That means "Events," "About," "Sponsors," etc. will never rank as their own separate search results the way separate pages on a normal multi-page site would; they'll only ever be found through the homepage. This is a structural site-architecture question (converting to real routed pages), not something Search Console or GA4 configuration can fix — flagging it here so it's a known tradeoff, not a surprise later. Not urgent, just worth knowing.

---

## Quick reference — what needs Liam's Google account vs. what's already in the repo

| Task | Where | Status |
|---|---|---|
| `llms.txt` | Repo | ✅ Done |
| `sitemap.xml` | Repo | ✅ Done |
| GTM snippet in HTML | Repo | ✅ Done (placeholder ID) |
| Virtual pageview tracking | Repo | ✅ Done |
| GA4 property creation | Google Analytics | ⬜ Liam |
| GTM container creation | Google Tag Manager | ⬜ Liam |
| Swap placeholder GTM ID in HTML | Repo | ⬜ Liam (one-line edit, then redeploy) |
| GA4 Config tag + Virtual Pageview tag/trigger | GTM | ⬜ Liam |
| Search Console verification + sitemap submission | Search Console | ⬜ Liam |
