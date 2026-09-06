# Deploying morristownimprov.com

*The site auto-deploys from GitHub — there is no manual upload step. This note explains the flow and how to check or roll back a deploy.*

## How deploys work

The live site is hosted on **Cloudflare Pages**, connected directly to this GitHub repo. Every push to the `main` branch triggers an automatic deployment — no build step, no manual file upload. The site is plain static files (`index.html` + `images/`), served as-is.

**The whole flow to publish a change:**

```
git add <files>
git commit -m "your message"
git push
```

Once the push lands on GitHub, Cloudflare picks it up and deploys automatically. The live site (morristownimprov.com) updates within about a minute, occasionally a couple of minutes.

## If a change doesn't appear

Work down this list — it's almost always one of the first two:

1. **Did the push actually land?** Run `git status` — if it says `ahead of 'origin/main'`, the commit is still only local. Run `git push`.
2. **Browser / CDN cache.** Cloudflare and your browser cache aggressively. Hard-refresh with **Ctrl+F5** (or open the site in a private window). This resolves the large majority of "it didn't update" cases.
3. **Deployment still building or failed.** Check the Cloudflare dashboard (below).

## Checking a deploy in Cloudflare

1. Go to [dash.cloudflare.com](https://dash.cloudflare.com) and log in.
2. **Workers & Pages** → open the morristownimprov.com project.
3. The **Deployments** list shows every push, its status (Building / Success / Failed), and the commit it came from. The top successful one is what's live.
4. Each deployment has its own preview URL if you want to check a build before it's promoted.

## Rolling back a bad deploy

If a deploy breaks the site, you don't have to fix-forward under pressure:

- In the **Deployments** list, find the last known-good deployment and use **Rollback** (the "..." / actions menu on that deployment) to make it live again immediately.
- Then fix the problem in the repo and push again at your own pace.

## Verify after a deploy (on morristownimprov.com itself)

- All pages load: Home, Events, About, Team, Get Involved, Sponsors, Gallery, Book a Show (hash-routed — `/#events`, `/#team`, etc. should work directly, including a hard refresh or a pasted link).
- Images render on every page, including Gallery and Team.
- Nav links, the "Book a Show" request form, and the survey all work.
- Favicon shows in the browser tab.

## Notes

- **Domain/DNS** (Spaceship registrar, Cloudflare nameservers) needs no changes for a content update — that was one-time first-launch setup.
- The old manual "direct upload / Upload assets (not Git)" process this file used to describe is **obsolete** — the Git connection replaced it. Don't upload assets by hand; just push.
