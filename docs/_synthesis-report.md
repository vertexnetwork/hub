# Vertex Network — Cross-Audit Synthesis Report

> **Corpus.** 5 single-spoke audits in this directory:
> [captionsnap.md](captionsnap.md), [etsy-profit-calc.md](etsy-profit-calc.md), [kdpcover.md](kdpcover.md), [shopifont.md](shopifont.md), [tokenmath.md](tokenmath.md).
>
> **Method.** Union consensus — every recommendation from any audit graduates to the template. Cells marked ✓ = present in that spoke per its audit; **GAP** = audit explicitly flagged absent; — = audit didn't mention it (insufficient evidence to grade either way).
>
> **Outputs.** This report (Stay vs Go); [`_scaffold-spec.md`](_scaffold-spec.md) (the canonical spec); [`_canonical-audit-prompt.md`](_canonical-audit-prompt.md) (the re-usable audit prompt).

---

## TL;DR

1. **The chassis is ~70% extractable today.** Every audit independently arrived at this number. The shape is solid; the gaps are concentrated in three universal areas (below).
2. **The Big Three Universal Gaps** appear in 4–5 of 5 audits and become P0 template requirements: `siteConfig.ts` keystone, search-engine verification meta plumbing (GSC + Bing), multi-size favicon set + `manifest.webmanifest`.
3. **`/network` is the headline hub-and-spoke target.** Every spoke has a hardcoded sister-site array. Every audit identifies the same fix: central `network.json`, `repository_dispatch` fan-out via GitHub Actions, spoke commits the synced JSON, Vercel rebuilds.
4. **Static-export + ad provider + monetization choice are the genuine spoke-level divergences.** Everything else converges. Discrepancies in §I have prescribed resolutions.
5. **Manual config is mostly automatable.** Of 11 manual launch steps, 7 can be scripted (Vercel project creation, env seeding, DNS via Namecheap API, sitemap pings, favicon generation, redirect config, repo creation). The 4 that cannot are AdSense/Mediavine application, Chrome Web Store submission, GSC/Bing site claim, and social handle reservation — these go in a launch checklist.

---

## The Big Three Universal Gaps

Called out by 4 or 5 of 5 audits. These are P0 for any new template scaffold and the highest-priority remediation work for existing spokes.

### 1. `siteConfig.ts` — the missing keystone

Every audit independently identified the same root problem: brand strings (name, domain, email, social handles, theme colors, nav links) are hardcoded across 8–15 files per spoke. Every audit proposed the same fix: a single `lib/site-config.ts` module, every other file reads from it.

| Spoke | Where the audit flagged it |
|---|---|
| captionsnap | §6 — "No `siteConfig.ts` exists. Centralize." (8+ hardcoded references) |
| etsy-profit-calc | §E — full proposed shape with `nav`, `theme`, `jsonLd`, `product`, `embed` |
| kdpcover | §3f, §9 step 1 — "Critical refactor for template" (10+ hardcoded references) |
| shopifont | §11 — "Drift Risks" section lists hardcoded contact email, GitHub URL, keywords, NAV_LINKS, success copy, lift to `lib/site.ts` |
| tokenmath | §A through §G — explicit "Section G — Hardcoded Brand-String Inventory" with 17 line-cited extraction points |

**Verdict:** **BUILD PROMPT** universal requirement. See spec §4 for the canonical shape.

### 2. Search-engine verification meta plumbing

Every audit flagged GSC + Bing verification as an explicit gap. The verification *codes* are per-site secrets but the *plumbing* (env-driven `<meta>` injection from root metadata) is identical.

| Spoke | GSC | Bing |
|---|---|---|
| captionsnap | GAP (§3c, §8) | GAP (§3c, §8) |
| etsy-profit-calc | GAP (§Section G, app/ section) | GAP |
| kdpcover | GAP (§4b, §5) | GAP |
| shopifont | GAP (§5 Missing) | GAP |
| tokenmath | GAP (§ ENV needed) | GAP |

**Verdict:** **BUILD PROMPT** universal requirement. Add `NEXT_PUBLIC_GOOGLE_SITE_VERIFICATION` and `NEXT_PUBLIC_BING_SITE_VERIFICATION` to `.env.example`; emit from root metadata when present. See spec §8 + §13.

### 3. Multi-size favicon set + `manifest.webmanifest`

Every audited spoke shipped only `app/icon.svg` (+ sometimes a 180px Apple touch icon via `next/og`). None had the full 16/32/192/512 PNG set required for PWA installability or platform-specific multi-size needs (Windows tiles, Android home-screen, Apple touch). Two spokes lacked `manifest.webmanifest` entirely.

| Spoke | Multi-size favicons | `manifest.webmanifest` |
|---|---|---|
| captionsnap | GAP | GAP |
| etsy-profit-calc | GAP | GAP |
| kdpcover | GAP | ✓ (`manifest.ts`) |
| shopifont | GAP | GAP |
| tokenmath | GAP | ✓ (`manifest.ts`) |

**Verdict:** **BUILD PROMPT** universal requirement. Ship `scripts/generate-favicon.ts` (sharp pipeline: SVG → 16/32/192/512 + apple-180) as part of the template. Generated files commit to `public/`. See spec §6.

---

## Stay vs Go matrix

### A. Universal infrastructure

| Asset | captionsnap | etsy | kdpcover | shopifont | tokenmath | Bucket |
|---|---|---|---|---|---|---|
| `robots.ts` | ✓ | ✓ | ✓ | ✓ | ✓ | **HUB** |
| `sitemap.ts` | ✓ | ✓ | ✓ | ✓ | ✓ | **HUB** (pattern) |
| `llms.txt` | ✓ | ✓ | ✓ | ✓ | ✓ | **HUB** (pattern) |
| `llms-full.txt` | ✓ | ✓ | ✓ | ✓ | ✓ | **HUB** (pattern) |
| `ads.txt` | GAP | ✓ | ✓ | GAP | ✓ | **BUILD PROMPT** |
| `app-ads.txt` | GAP | GAP | ✓ | GAP | ✓ | **BUILD PROMPT** |
| `security.txt` (RFC 9116) | GAP | GAP | GAP | GAP | GAP | **BUILD PROMPT** (universal gap) |
| `humans.txt` (optional) | GAP | GAP | GAP | GAP | GAP | **BUILD PROMPT** (optional) |
| `manifest.webmanifest` | GAP | GAP | ✓ | GAP | ✓ | **BUILD PROMPT** |
| Multi-size favicons (16/32/192/512) | GAP | GAP | GAP | GAP | GAP | **BUILD PROMPT** (universal gap) |
| Apple touch icon (180×180) | partial | ✓ | ✓ | GAP | ✓ | **BUILD PROMPT** |
| Dynamic OG route | ✓ | ✓ | ✓ | GAP | ✓ | **BUILD PROMPT** |
| Static `/og-default.png` fallback | GAP | — | GAP | GAP | — | **BUILD PROMPT** |
| GSC verification meta (env-driven) | GAP | GAP | GAP | GAP | GAP | **BUILD PROMPT** (universal gap) |
| Bing verification meta (env-driven) | GAP | GAP | GAP | GAP | GAP | **BUILD PROMPT** (universal gap) |
| `vercel.json` | GAP | GAP | GAP | ✓ | partial | **BUILD PROMPT** |
| `.nvmrc` | ✓ | — | GAP | GAP | — | **BUILD PROMPT** |
| `.gitattributes` (LF + binary) | — | — | — | — | ✓ | **BUILD PROMPT** |
| Dependabot config | GAP | — | GAP | — | ✓ | **BUILD PROMPT** |
| `CODEOWNERS` | GAP | — | — | — | ✓ | **BUILD PROMPT** |
| `ISSUE_TEMPLATE/` | GAP | — | — | — | — | **BUILD PROMPT** (optional) |
| `PULL_REQUEST_TEMPLATE.md` | — | — | — | — | ✓ | **BUILD PROMPT** (optional) |
| CI workflow (typecheck/lint/test) | ✓ | ✓ | GAP | ✓ | ✓ | **HUB** (reusable workflow `@v1`) |
| Lighthouse CI | — | — | — | ✓ | ✓ | **BUILD PROMPT** |

### B. Layout chrome

| Asset | captionsnap | etsy | kdpcover | shopifont | tokenmath | Bucket |
|---|---|---|---|---|---|---|
| Mobile hamburger header | GAP | ✓ | ✓ | ✓ | ✓ | **HUB** (4/5; explicit user requirement) |
| `SiteHeader` / `SiteFooter` | ✓ | ✓ | ✓ | ✓ | ✓ | **HUB** pattern (CONFIG-driven) |
| `/network` page | ✓ | ✓ | ✓ | ✓ | ✓ (footer modal) | **HUB** — *the centerpiece* |
| Tailwind v4 `@theme` tokens | ✓ | ✓ | ✓ | ✓ | ✓ | **HUB** pattern (per-site palette) |
| `siteConfig.ts` keystone | GAP | partial | GAP | partial | GAP | **BUILD PROMPT** (universal gap) |
| Sticky mobile CTA | — | — | — | ✓ | — | OPTIONAL |
| Theme toggle (dark/light) | — | — | — | — | ✓ | DISCREPANCY — opt-in |
| Cookie consent provider | ✓ (full) | — | — | — | — | DISCREPANCY — adopt for EU traffic |

### C. Analytics

| Asset | captionsnap | etsy | kdpcover | shopifont | tokenmath | Bucket |
|---|---|---|---|---|---|---|
| Vercel Analytics | ✓ | ✓ | ✓ | ✓ | ✓ | **HUB** |
| Vercel Speed Insights | — | — | ✓ | ✓ | ✓ | **BUILD PROMPT** |
| Microsoft Clarity (env-gated) | ✓ | ✓ | ✓ | ✓ | ✓ | **HUB** |
| Plausible | — | — | — | ✓ | — | OPTIONAL |
| GA4 / gtag | — | ✓ | — | — | — | OPTIONAL |

### D. SEO / JSON-LD

| Asset | captionsnap | etsy | kdpcover | shopifont | tokenmath | Bucket |
|---|---|---|---|---|---|---|
| `JsonLd` component | ✓ | ✓ | ✓ | ✓ | ✓ | **HUB** |
| `WebSite` / `WebApplication` | ✓ | ✓ | ✓ | ✓ | ✓ | **HUB** |
| `Organization` schema | — | — | ✓ | ✓ | ✓ | **HUB** |
| `BreadcrumbList` | ✓ | — | ✓ | ✓ | ✓ | **HUB** |
| `FAQPage` schema | — | ✓ | — | ✓ | ✓ | **HUB** |
| `HowTo` schema | — | — | ✓ | ✓ | — | OPTIONAL |
| `Article` schema | ✓ | — | — | ✓ | — | OPTIONAL |
| `AboutPage` (Mediavine req) | — | — | — | ✓ | — | **BUILD PROMPT** (Mediavine sites only) |
| `SoftwareApplication` / `FinanceApplication` | ✓ | ✓ | ✓ | ✓ | ✓ | **HUB** (parameterized `@type`) |
| pSEO `[slug]/page.tsx` pattern | ✓ (101 pages) | ✓ | ✓ | ✓ (71 pages) | ✓ (10 models) | **HUB** (highest-leverage pattern) |
| AI-bot allowlist in `robots` | — | — | — | — | ✓ | **HUB** (sync from `vertex-hub/ai-bots.json`) |

### E. Monetization (per-site, mostly OPTIONAL)

| Asset | captionsnap | etsy | kdpcover | shopifont | tokenmath | Bucket |
|---|---|---|---|---|---|---|
| Stripe subscriptions | ✓ | — | — | — | — | OPTIONAL pattern |
| Lemon Squeezy | — | — | ✓ | — | — | OPTIONAL pattern |
| Gumroad | — | ✓ | — | — | — | OPTIONAL pattern |
| AdSense (env-gated) | — | ✓ | — | ✓ | ✓ | OPTIONAL |
| Mediavine (env-gated) | — | ✓ | — | ✓ | ✓ | OPTIONAL |
| Carbon Ads (env-gated) | — | — | — | — | ✓ | OPTIONAL |
| `AdSlot` abstraction (CLS-safe) | — | ✓ | — | ✓ | ✓ | **HUB** (when ads on) |
| Affiliate slot/cards | ✓ | ✓ | ✓ | ✓ | ✓ | **HUB** (pattern, env-driven) |
| Email capture (Resend) | — | — | — | ✓ | — | OPTIONAL |
| Embed widget + CSP `frame-ancestors *` | ✓ | ✓ | ✓ | ✓ | — | **BUILD PROMPT** (opt-in) |
| Chrome extension companion | ✓ | ✓ | ✓ | ✓ | — | OPTIONAL pattern |

### F. CI/CD & repo hygiene

| Asset | Recommended in | Bucket |
|---|---|---|
| Husky / lint-staged pre-commit | captionsnap, kdpcover | **BUILD PROMPT** (optional) |
| Auto-changelog from `git log` (`prebuild`) | shopifont, etsy, kdpcover, tokenmath (4/5) | **HUB** |
| Sitemap-ping on deploy (GSC IndexNow + Bing) | etsy, kdpcover | **BUILD PROMPT** |
| Broken-link cron | etsy, kdpcover | **BUILD PROMPT** (optional) |
| OG image snapshot test | etsy | **BUILD PROMPT** (optional) |
| A11y audit (axe-core) | etsy | **BUILD PROMPT** (optional) |
| Favicon generation pipeline (SVG → multi-size) | etsy, kdpcover, tokenmath | **HUB** (script) |
| Network-sync workflow (`repository_dispatch`) | All 5 | **HUB** (the headline automation) |

---

## Hub-and-Spoke roadmap

### Sync targets (consensus)

All 5 audits propose the same Tier-1 hub artifacts:

| # | Asset | Source-of-truth (hub) | Spoke artifact | Consumed by |
|---|---|---|---|---|
| 1 | Network registry | `vertex-hub/config/network.json` | `public/network.json` (synced) | `/network` page, sitemap, JSON-LD `sameAs`, llms.txt sister-site index |
| 2 | Network philosophy prose | `vertex-hub/content/network-philosophy.mdx` | `content/network-philosophy.mdx` | `/network` page |
| 3 | `ads.txt` / `app-ads.txt` | `vertex-hub/crawler/{ads,app-ads}.txt` | `public/{ads,app-ads}.txt` | served at `/ads.txt`, `/app-ads.txt` |
| 4 | `security.txt` | `vertex-hub/crawler/security.txt` (template) | `public/.well-known/security.txt` | served per RFC 9116 |
| 5 | AI bot allowlist | `vertex-hub/config/ai-bots.json` | `public/ai-bots.json` | `app/robots.ts` |
| 6 | Reusable CI workflow | `vertex-network/.github/workflows/ci.yml@v1` | `uses:` reference in spoke `ci.yml` | every PR |
| 7 | Privacy/Terms boilerplate | `vertex-hub/content/legal/{privacy,terms}.mdx` (with `{{placeholders}}`) | rendered into `app/(marketing)/{privacy,terms}/page.tsx` | legal pages |
| 8 | CSP provider allowlist | `vertex-hub/config/csp-providers.json` | consumed by `lib/csp.ts` | `next.config.ts` headers |
| 9 | Cookie consent strings | `vertex-hub/config/consent-strings.json` | bundled JSON | `<CookieConsent>` |
| 10 | Recommended-tools / cross-affiliate | `vertex-hub/config/recommended-tools.json` | `data/recommended-tools.json` | `/recommendations` page (when applicable) |

### Sync mechanism (recommended)

**PR-based fan-out via `repository_dispatch`**, per consensus across kdpcover §7 (Option B), shopifont §10, etsy §F.3, tokenmath §F.1 (Option A):

1. Hub `propagate.yml` triggers on push to `main` to any synced file.
2. Reads `spokes.json` registry (list of spoke repos).
3. For each spoke, dispatches `repository_dispatch` with payload listing changed files.
4. Spoke's `sync-from-hub.yml` listens, runs a sync script (writes the changed files), opens an auto-merge PR via `peter-evans/create-pull-request`.
5. Spoke CI gates the merge; auto-merges if green.
6. Vercel auto-deploys on merge.

Why PR-based over build-time fetch: every change is auditable per-spoke; spokes keep building if hub is briefly unavailable; fan-out is atomic and observable.

### Recommended hub repo structure

Converged from etsy §H, kdpcover §10, shopifont §12 ("vertex-network-meta"), tokenmath §H. Verbatim definition is in `_scaffold-spec.md` §12.

---

## Manual-steps automation roadmap

| Step | Mentioned in | Automation path | Blocker? |
|---|---|---|---|
| Domain registration (Namecheap) | tokenmath, etsy | Manual buy or Namecheap API in scaffolder | No |
| DNS A/CNAME | tokenmath, etsy | Namecheap API in scaffolder | No |
| Vercel project creation | etsy, tokenmath | Vercel API in scaffolder | No |
| Vercel env var seeding | All 5 | Vercel API in scaffolder | No |
| Apex ↔ www redirect | kdpcover, shopifont | `vercel.json` template (no API needed) | No |
| GSC site claim | All 5 | Manual (Google's UI requires human verification) | **Yes** |
| GSC verification meta wiring | All 5 | env-driven; **automatable plumbing** | No |
| GSC sitemap submission on deploy | etsy, kdpcover | IndexNow API in `sitemap-ping.yml` | No |
| Bing site claim | All 5 | Manual (Bing's UI) | **Yes** |
| Bing verification meta wiring | All 5 | env-driven; **automatable plumbing** | No |
| Bing IndexNow ping on deploy | etsy, kdpcover | IndexNow API in `sitemap-ping.yml` | No |
| AdSense site claim | etsy | Manual application | **Yes** |
| Mediavine application | etsy | Manual (traffic threshold required) | **Yes** |
| Chrome Web Store submission | etsy | Manual review process | **Yes** |
| Social handle reservation | etsy | Manual | **Yes** |
| Favicon generation (SVG → multi-size) | etsy, kdpcover, tokenmath | `scripts/generate-favicon.ts` (sharp) | No |
| Auto-changelog from `git log` | 4/5 | `scripts/build-changelog.ts` in `prebuild` | No |
| Network registry update | All 5 | Hub `repository_dispatch` fan-out | No |
| Screenshot bot for `/network` | kdpcover | Optional GH Action on main push | No |

**11 of 19 steps are fully automatable.** The 5 hard blockers (GSC/Bing claim, AdSense/Mediavine application, Chrome Web Store, social handles) live in a launch checklist (`vertex-hub/docs/SITE_LAUNCH_CHECKLIST.md`).

---

## Per-site env var canonical list

Union of all 5 audits' `.env.example` proposals. Verbatim list lives in `_scaffold-spec.md` §13. Highlights:

- **6 identity vars** (name, short_name, domain, url, description, contact_email) — universal, hub-mandated additions.
- **2 SEO verification vars** (Google, Bing) — universal, env-driven plumbing fixes the universal gap.
- **3 analytics vars** (Vercel toggle, Clarity, optional Plausible) — env-gated.
- **5 ad-stack vars** (provider switch + AdSense client + Mediavine site + 2 Carbon vars) — opt-in.
- **3 affiliate vars** (url + label + provider) — generic (replaces tokenmath's `RUNPOD_REF_URL`-style per-site naming).
- **6+ monetization vars** (Stripe / Lemon Squeezy / Gumroad / Resend / Embed) — opt-in via `siteConfig.features.*`.

---

## GitHub Actions automation tiers

Tiered by effort, converged from etsy §F, kdpcover §8, shopifont §10, tokenmath §F.

### Tier 1 — ship in template day one

1. `ci.yml` — typecheck / lint / vitest / build (calls hub reusable workflow).
2. `sync-from-hub.yml` — `repository_dispatch` listener.
3. `lighthouse.yml` — LHCI on preview deploys (mobile budgets perf≥0.9, a11y/bp/seo≥0.95, **CLS=0**).
4. `validate-seo-artifacts.yml` — fetch built sitemap/robots/ads.txt/llms.txt; lint structure (xmllint, IAB regex).
5. `generate-favicon-set.yml` — on `app/icon.svg` change, run sharp pipeline → commit PNGs.
6. `dependabot.yml` — npm + GH Actions weekly grouped updates.
7. `vercel-deploy-preview-comment.yml` — preview URL on PR.

### Tier 2 — per-site rollout as needed

8. `sitemap-ping.yml` — POST sitemap to GSC IndexNow + Bing IndexNow on production deploys.
9. `og-image-snapshot-test.yml` — Playwright pixel-diff vs golden.
10. `broken-link-audit.yml` — weekly `lychee` over deployed site + cross-network outbound.
11. `changelog-coverage.yml` — every commit since last release reflected in `content/changelog.json`.
12. `a11y-audit.yml` — axe-core CI on home + sample pSEO page.

### Tier 3 — hub-level

13. `propagate.yml` (hub) — fan-out `repository_dispatch` on hub change.
14. `network-graph-rebuild.yml` (hub) — on `network.json` change, immediately fan out instead of waiting for cron.
15. `scheduled-lighthouse-monitor.yml` (hub) — weekly LHCI run against all production network sites; results checked into `vertex-hub/metrics/`; regressions open issues in affected spokes.
16. `screenshot-bot.yml` — on main push, capture homepage, commit to hub `screenshots/{domain}.png` for `/network` previews.
17. `data-refresh-template.yml` — parameterized version of tokenmath's pricing-refresh pattern (vendor scrape → Claude Haiku JSON extraction → sanity-gate → auto-PR).

---

## Discrepancies (audits disagree)

| # | Item | Disagreement | Resolution per spec |
|---|---|---|---|
| 1 | Cookie consent | captionsnap has full ConsentProvider; others silent | **Adopt** network-wide for EU traffic (`siteConfig.features.consent.required = true`). Spec §7. |
| 2 | Theme toggle (dark/light) | Only tokenmath has it | **Optional** — `siteConfig.features.themeToggle`. Spec §7. |
| 3 | `vercel.json` content | shopifont rich (HSTS, X-Frame, cache); kdpcover none; tokenmath in `next.config.ts` | **Standardize** in `vercel.json` per spec §6. Mirror security headers in `next.config.ts` only when static export disallows. |
| 4 | `output: 'export'` static export | shopifont uses; others SSR/ISR | **Per-site architectural choice.** Template ships SSR-default with comments showing how to flip. Spec §2. |
| 5 | Embed widget | 4/5 yes; tokenmath no | **Opt-in** via `siteConfig.features.embed.enabled`. Template ships infrastructure (CSP, route shell), site decides. Spec §10. |
| 6 | Chrome extension | 4/5 yes; tokenmath no | **Opt-in** via `siteConfig.features.extension.enabled`. Hub ships `templates/extension/` Vite MV3 starter. Spec §3. |
| 7 | Affiliate count | tokenmath = 1 enforced; others = many | **Network principle**, not enforced in code. Documented in `vertex-hub/docs/ARCHITECTURE.md`. |
| 8 | Lighthouse CI | shopifont + tokenmath yes; others no | **Adopt** at STANDARD compliance. Spec §11, §15. |
| 9 | Plausible vs Vercel-only analytics | Only shopifont uses Plausible | **Optional**. Vercel Analytics is default-on; Plausible env-gated. Spec §9. |
| 10 | pSEO scale (10–101 entries) | Different per spoke | **Per-site.** Pattern is HUB; data is SPOKE. Spec §8. |

---

## Architectural direction (consensus)

All 5 audits independently converge on:

- **Hub repo with `network.json`, `ads.txt`, `security.txt`, legal MDX, AI bot list**, propagated via `repository_dispatch`.
- **Reusable GitHub Actions workflows** versioned `@v1` so breaking changes don't auto-roll spokes.
- **`siteConfig.ts`** as the per-site keystone (universal gap, must be added).
- **pSEO `[slug]` route as the universal content-multiplier** — the highest-leverage pattern in the network. Recipe: data source → MDX file → dynamic route + `generateStaticParams` + `generateMetadata` → sitemap → llms.txt.
- **Tailwind v4 `@theme` block as the rebrand entry point** — strongest part of the chassis. Token names canonical; values per-site.
- **Versioned `@vertex/*` shared packages as a future option** (etsy §C, tokenmath §H). Not required at template v1; storage and changelog are smallest-blast-radius candidates if pursued.

---

## Verification

When the synthesis is implemented:

1. **Cross-check completeness.** Every cell in matrix sections A–F maps to a section in `_scaffold-spec.md`. No orphans.
2. **Round-trip audit.** Run `_canonical-audit-prompt.md` against `c:\Users\joshu\OneDrive\Desktop\captionsnap` and compare its P0 output to [captionsnap.md](captionsnap.md) §12 ("Gap Summary"). The two punch lists should match.
3. **Empty-project test.** Run the prompt against an empty directory; output should read as a complete build spec (every spec section appears as a P0 GAP).
4. **Template scaffold pilot.** Bootstrap a new site from the spec; edit only `siteConfig.ts` + `app/icon.svg` + `globals.css` `@theme` palette + the hero feature; confirm sitemap renders with correct domain, `/network` shows sister sites, JSON-LD validates at validator.schema.org, Lighthouse SEO ≥95.
5. **Hub fan-out pilot.** Stand up `vertex-hub/config/network.json` (seeded from any existing spoke's network array), wire `propagate.yml` + spoke `sync-from-hub.yml`, change `network.json`, confirm PR appears in spoke within 2 minutes.
6. **Compliance grading.** Run prompt at MINIMAL / STANDARD / PREMIUM levels against the same spoke; confirm grade strictly increases in difficulty and that PREMIUM-only items (sitemap-ping, broken-link cron, axe-core, OG snapshot, screenshot bot) appear only at PREMIUM.

---

## Out of scope (downstream of this synthesis)

These are the next concrete moves, but not built in this exercise:

1. Stand up `vertex-hub` and `vertex-network/.github` repos.
2. Author `propagate.yml` (hub) + `sync-from-hub.yml` (spoke) workflows.
3. Build `siteConfig.ts` shape; refactor one existing spoke (suggested: tokenmath, smallest divergence) to consume it as the validation case.
4. Add the Big Three universal gaps (`siteConfig.ts`, GSC+Bing meta, multi-size favicons) to every existing spoke.
5. Promote `ci.yml` to reusable workflow at `vertex-network/.github/workflows/ci.yml@v1`.
6. Build `create-vertex-site` scaffolder using the now-templatized files.
7. Migrate next spoke onto the template; treat divergences as backlog for the next analyzer pass.
