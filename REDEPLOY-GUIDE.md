# Redeploying morristownimprov.com — Direct Upload

*For updating the existing Cloudflare Pages project with this zip. Not a first-time setup — the site and custom domain are already live; this just pushes new content to them.*

## What's in this zip

`index.html` + `images/` — the entire site, ready to upload as-is. No build step, no config.

## Steps

1. Extract the zip on your computer. `index.html` and `images/` should be sitting next to each other after extracting — that's what needs to end up at the root of the upload.
2. Go to [dash.cloudflare.com](https://dash.cloudflare.com) and log in.
3. **Workers & Pages** → open the existing project for morristownimprov.com (not "Create application" — this project already exists).
4. Inside the project, find the option to upload a new deployment — depending on the current dashboard version this is either a **Deployments** tab → **Create deployment**, or an **Upload assets** button. Choose **Upload assets** / direct upload (not Git).
5. Select the *contents* of the extracted folder — `index.html` and `images/` need to be selected directly, not a parent folder containing them. If `index.html` isn't visible at the root of what's being uploaded, the deploy will 404.
6. Leave build command and build output directory blank — this is a static asset upload, not a build.
7. Deploy. Cloudflare gives an immediate `*.pages.dev` preview URL — check that first.
8. Since morristownimprov.com is already connected to this project, the new deployment goes live on the real domain automatically once it's the active deployment — no DNS or domain step needed this time.

## Verify after deploy (on morristownimprov.com itself, not just the preview URL)

- All pages load: Home, Events, About, Team, Get Involved, Sponsors, Gallery, Book a Show (hash-routed — `/#events`, `/#team`, etc. should all work directly, including a hard refresh or a pasted link).
- Images render on every page, including the new Gallery and Team pages.
- Nav links, the "Book a Show" request form, and the survey all work as expected.
- Favicon shows in the browser tab.

## Common gotcha

**Broken images or a blank page after deploy** — almost always means the zip's inner folder got uploaded instead of its contents. `index.html` must sit at the root of what Cloudflare receives, with `images/` as a sibling folder, not one level deeper.

No domain, DNS, or nameserver changes are needed for this — that part was only relevant the first time the site went live.
