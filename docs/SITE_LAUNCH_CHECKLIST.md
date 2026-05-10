# SITE_LAUNCH_CHECKLIST

The per-spoke runbook. Use this when bringing a brand-new site online OR when migrating an existing spoke onto the canonical spec.

Steps marked **[manual]** can't be automated end-to-end. Steps marked **[scriptable]** are candidates for the future `create-vertex-site` CLI.

---

## Before you start

- New repo created on GitHub (or existing one to migrate).
- Domain registered at Namecheap.
- Vercel account ready.

## Phase A — Repo skeleton

1. **[scriptable]** Clone the spoke repo locally.
2. **[scriptable]** Copy every file under `templates/spoke/` from the hub into the spoke. Preserve the directory structure.
3. **[scriptable]** Create `lib/site-config.ts` per [`_scaffold-spec.md`](_scaffold-spec.md) §4. Fill in this site's name, domain, theme colors, etc.
4. **[scriptable]** Create `app/globals.css` with `@theme` token block per spec §5; tweak palette to match this site's brand.
5. **[scriptable]** Create the six required pages (per spec §7 "Required pages"): `/about`, `/contact`, `/privacy`, `/terms`, `/changelog`, `/network`.
6. **[scriptable]** Add `scripts/build-changelog.ts`, `scripts/generate-favicon.ts` per spec §3.
7. **[scriptable]** Drop `app/icon.svg` (your 64×64 brand mark) and run `pnpm generate:favicon` to produce the multi-size set.
8. Initial commit + push.

## Phase B — Vercel + DNS

9. **[scriptable]** Create Vercel project, link the GitHub repo (Vercel API or dashboard).
10. **[scriptable]** Add the domain to the Vercel project. Vercel will display the required A record (`76.76.21.21`) and CNAME (`cname.vercel-dns.com`).
11. **[manual]** In Namecheap → Domain → Advanced DNS:
    - Add `A` record: `@` → `76.76.21.21`
    - Add `CNAME` record: `www` → `cname.vercel-dns.com`
    - Wait for DNS propagation (usually 5–30 min).
12. **[scriptable]** Seed Vercel env vars from `.env.example` (Vercel CLI: `vercel env add`). Use the canonical list in [`config/env.template.json`](../config/env.template.json) as the source of truth.
13. Verify `https://yourdomain.com` resolves and the site loads.

## Phase C — Search engine claims **[manual]**

14. **[manual]** **Google Search Console** → https://search.google.com/search-console
    - Add property → URL prefix → `https://yourdomain.com`
    - Choose **HTML tag** verification.
    - Copy the verification token.
    - Add `NEXT_PUBLIC_GOOGLE_SITE_VERIFICATION=<token>` to Vercel env vars.
    - Redeploy. Click **Verify** in GSC.
    - Submit `https://yourdomain.com/sitemap.xml`.
15. **[manual]** **Bing Webmaster Tools** → https://www.bing.com/webmasters
    - Add site → choose **Meta tag** verification.
    - Copy the `msvalidate.01` content value.
    - Add `NEXT_PUBLIC_BING_SITE_VERIFICATION=<token>` to Vercel env vars.
    - Redeploy. Verify in Bing.
    - Submit sitemap.

## Phase D — Hub registration

16. **[scriptable]** Open `vertex-network-hub/config/network.json`. Add an entry for this site (`Property` shape per spec §7).
17. **[scriptable]** Open `vertex-network-hub/config/spokes.json`. Add the spoke's `<owner>/<repo>`.
18. Commit + push the hub. `propagate.yml` fans the new entry out to every existing spoke.
19. The new spoke gets `sync-from-hub.yml` installed (Phase A step 2). On the next hub push, it syncs in `network.json` and starts seeing siblings.

## Phase E — Analytics

20. **[manual]** **Microsoft Clarity** (optional) → https://clarity.microsoft.com → create project for this domain → copy project ID → add `NEXT_PUBLIC_CLARITY_PROJECT_ID=<id>` to Vercel.
21. Vercel Analytics + Speed Insights work automatically once the project is on Vercel — no additional setup.

## Phase F — Monetization (only if applicable)

22. **[manual]** **AdSense** → https://www.google.com/adsense → apply for the site → wait for approval (1–14 days) → once approved, add `NEXT_PUBLIC_AD_PROVIDER=adsense` and `NEXT_PUBLIC_ADSENSE_CLIENT_ID=<pub-id>` to Vercel.
23. **[manual]** **Mediavine Journey** (alternative) → https://www.mediavine.com/journey → apply once site has 50K+ monthly sessions → switch `NEXT_PUBLIC_AD_PROVIDER=mediavine`.
24. **[manual]** **Stripe** (Pro tier sites) → https://dashboard.stripe.com → create product + prices → add `STRIPE_*` vars to Vercel → set `NEXT_PUBLIC_PRO_ENABLED=1`.
25. **[manual]** **Resend** (email capture sites) → https://resend.com → verify sending domain → create audience → add `RESEND_API_KEY` and `RESEND_AUDIENCE_ID` to Vercel.

## Phase G — Validation

26. Visit `https://yourdomain.com/robots.txt` — should list AI bots from `public/ai-bots.json`.
27. Visit `https://yourdomain.com/sitemap.xml` — should resolve.
28. Visit `https://yourdomain.com/llms.txt` and `/llms-full.txt` — should resolve.
29. Visit `https://yourdomain.com/ads.txt` and `/app-ads.txt` — should resolve (synced from hub).
30. Visit `https://yourdomain.com/.well-known/security.txt` — should resolve.
31. Visit `https://yourdomain.com/network` — should list every other site in the network (this one omitted).
32. Run [Schema.org validator](https://validator.schema.org/) against the home page — JSON-LD should be valid.
33. Run [Lighthouse](https://pagespeed.web.dev/) → Mobile → all four scores ≥ 95, CLS = 0.
34. Click "Part of the Vertex Network" in the footer — should land on `/network`.
35. Open mobile view (DevTools → 375px) — hamburger menu should work, Esc should close it.

If any of 26–35 fail, fix before announcing the site.

## Phase H — Optional / nice-to-haves

- **[manual]** Reserve the social handle (Twitter/X, etc.).
- **[manual]** Submit Chrome extension to the Web Store (if applicable).
- **[scriptable]** Set up `screenshot-bot.yml` to commit a homepage screenshot to `vertex-network-hub/screenshots/<domain>.png` for `/network` previews.
- **[scriptable]** Wire `sitemap-ping.yml` to POST sitemap to GSC IndexNow and Bing IndexNow on production deploys.

---

**Time budget**: ~2 hours per spoke if you've done it once. ~30 minutes once `create-vertex-site` exists.
