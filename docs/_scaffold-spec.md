# Vertex Network — Canonical Spoke Spec

> **Authority.** This document is the source of truth for what a Vertex Network spoke site looks like. The audit prompt at `_canonical-audit-prompt.md` grades projects against this spec. Synthesized from 5 cross-spoke audits (`captionsnap.md`, `etsy-profit-calc.md`, `kdpcover.md`, `shopifont.md`, `tokenmath.md`) using **union** consensus — every recommendation from any audit graduates here.
>
> **Vocabulary used throughout this spec:**
> - **HUB** — file/value lives in the central `vertex-hub` repo and is propagated to spokes
> - **SPOKE** — file/value lives only in the per-site repo
> - **ENV** — sourced from `process.env.NEXT_PUBLIC_*`
> - **CONFIG** — sourced from `lib/site-config.ts`
> - **GENERATED** — emitted by a build script (`prebuild`)

---

## 1. Mission

Build indie-tool sites with **chassis-identical** infrastructure and **product-unique** value props. Every site ships:

- Hosting: GitHub → Vercel, Namecheap apex domain (with `www` 308 → apex)
- Chrome: hamburger mobile header, footer with blind link to "Part of the Vertex Network", `/network` page
- SEO/GEO surface: `robots.ts`, `sitemap.ts`, `llms.txt`, `llms-full.txt`, `ads.txt`, `app-ads.txt`, `security.txt`
- Analytics: Vercel Analytics + Speed Insights, Microsoft Clarity (env-gated), GSC + Bing verification (env-driven)
- Optional monetization: env-gated ad provider switch (Google AdSense to start, switch to MediaVine Journey) + single affiliate slot + optional Stripe/Lemon Squeezy/Gumroad

The hub-and-spoke goal: a single source of truth in `vertex-hub` propagates network-wide updates via GitHub Actions; spokes contain only what's genuinely site-specific.

---

## 2. Stack (locked, T1)

| Layer | Pin |
|---|---|
| Runtime | Node 22 (`.nvmrc`) |
| Package manager | pnpm (workspace if extension/sibling apps) |
| Framework | Next.js 15+ App Router |
| UI | React 19 |
| Language | TypeScript 5+ strict, `@/*` path alias |
| Styling | Tailwind v4 via `@tailwindcss/postcss` (CSS-first; **no `tailwind.config.*`**) |
| Testing | Vitest (unit/jsdom) + Playwright (e2e) |
| Linting | ESLint 9 flat config (`next/core-web-vitals` + `next/typescript` + `prettier`) |
| Formatting | Prettier 3 (100 col, double-quote, trailing-comma all) |
| MDX | `@next/mdx` + `remark-gfm` + `rehype-slug` + `rehype-autolink-headings` (when MDX is used) |
| Hosting | Vercel (auto-deploy on push to `main`) |

Static export (`output: 'export'`) is **per-site optional** — most spokes use SSR/ISR.

---

## 3. Canonical repo layout

```
<spoke>/
├── .github/
│   ├── workflows/
│   │   ├── ci.yml                          # uses: vertex-network/.github/workflows/ci.yml@v1
│   │   ├── lighthouse.yml                  # uses: vertex-network/.github/workflows/lighthouse.yml@v1
│   │   ├── sync-from-hub.yml               # repository_dispatch listener
│   │   └── sitemap-ping.yml                # post-deploy GSC IndexNow + Bing
│   ├── CODEOWNERS
│   ├── PULL_REQUEST_TEMPLATE.md
│   ├── ISSUE_TEMPLATE/
│   │   ├── bug.md
│   │   └── feature.md
│   └── dependabot.yml
├── app/
│   ├── layout.tsx                          # CONFIG-driven metadata, fonts, providers
│   ├── globals.css                         # @theme tokens, palette per spoke
│   ├── page.tsx                            # SPOKE — hero/product
│   ├── (marketing)/                        # OR (site)/ — chrome-wrapped routes
│   │   ├── layout.tsx                      # SiteHeader + main + SiteFooter
│   │   ├── about/page.tsx                  # REQUIRED — SPOKE prose
│   │   ├── changelog/page.tsx              # REQUIRED — GENERATED, dates + titles only (no full notes)
│   │   ├── contact/page.tsx                # REQUIRED — CONFIG-driven (email)
│   │   ├── network/page.tsx                # REQUIRED — HUB, reads public/network.json
│   │   ├── privacy/page.tsx                # REQUIRED — HUB MDX with {{placeholders}} OR CONFIG
│   │   ├── terms/page.tsx                  # REQUIRED — SPOKE
│   │   └── pricing/page.tsx                # SPOKE — gated by features.proEnabled
│   ├── (pseo)/[noun]/[slug]/page.tsx       # SPOKE pattern, HUB shape
│   ├── embed/                              # OPT-IN (features.embed.enabled)
│   │   ├── layout.tsx
│   │   └── page.tsx
│   ├── api/
│   │   ├── og/route.tsx                    # Edge ImageResponse, CONFIG-driven
│   │   ├── subscribe/route.ts              # OPT-IN (features.email.enabled, Resend)
│   │   └── stripe/                         # OPT-IN (features.proEnabled, Stripe)
│   ├── robots.ts                           # ENV+HUB (ai-bots from public/ai-bots.json)
│   ├── sitemap.ts                          # CONFIG+SPOKE (multi-shard if >45k URLs)
│   ├── manifest.ts                         # CONFIG-driven
│   ├── icon.svg                            # SPOKE — brand mark
│   ├── apple-icon.tsx                      # CONFIG-driven (next/og)
│   ├── opengraph-image.tsx                 # CONFIG-driven (next/og) — root fallback
│   ├── twitter-image.tsx                   # re-exports opengraph-image
│   └── not-found.tsx                       # CONFIG-driven 404
├── components/
│   ├── layout/
│   │   ├── SiteHeader.tsx                  # HUB pattern, CONFIG-driven nav, hamburger required
│   │   ├── SiteFooter.tsx                  # HUB pattern, CONFIG-driven, "Part of the Vertex Network"
│   │   ├── MobileMenu.tsx                  # HUB pattern (hamburger drawer, Esc-to-close)
│   │   └── StickyMobileCta.tsx             # OPT-IN
│   ├── analytics/
│   │   ├── Analytics.tsx                   # Vercel Analytics + Speed Insights
│   │   └── Clarity.tsx                     # ENV-gated MS Clarity
│   ├── ads/
│   │   ├── AdsProviderScript.tsx           # ENV switch: none|adsense|mediavine|carbon
│   │   ├── AdSlot.tsx                      # CLS-safe height reserve via --ad-slot-*-h
│   │   ├── adsense.tsx
│   │   ├── mediavine.tsx
│   │   └── carbon.tsx
│   ├── seo/
│   │   ├── JsonLd.tsx                      # generic emitter
│   │   ├── SiteSchema.tsx                  # WebSite + Organization
│   │   ├── SoftwareApplicationSchema.tsx   # parameterized @type
│   │   ├── BreadcrumbSchema.tsx
│   │   ├── FaqSchema.tsx
│   │   ├── ArticleSchema.tsx               # OPT-IN
│   │   ├── HowToSchema.tsx                 # OPT-IN
│   │   └── AboutPageSchema.tsx             # required if Mediavine
│   ├── consent/
│   │   ├── ConsentProvider.tsx             # localStorage-backed
│   │   └── CookieConsent.tsx               # banner + Clarity gate
│   ├── affiliate/
│   │   └── AffiliateSlot.tsx               # ENV-driven generic slot
│   ├── email/                              # OPT-IN (features.email)
│   │   └── EmailCaptureForm.tsx
│   ├── pro/                                # OPT-IN (features.proEnabled, Stripe)
│   │   ├── ProProvider.tsx
│   │   ├── SubscribeButtons.tsx
│   │   └── AccountDashboard.tsx
│   ├── brand/
│   │   ├── Wordmark.tsx                    # CONFIG-driven aria-label
│   │   └── Logo.tsx                        # SPOKE SVG
│   └── mdx/MdxComponents.tsx               # design-token references only, no raw colors
├── content/
│   ├── changelog.json                      # GENERATED by scripts/build-changelog
│   ├── pseo/*.mdx                          # SPOKE
│   └── network-philosophy.mdx              # HUB (synced from vertex-hub)
├── lib/
│   ├── site-config.ts                      # THE KEYSTONE — see §4
│   ├── csp.ts                              # buildCSP({ providers }) — see §11
│   ├── network.ts                          # loads public/network.json
│   ├── seo.ts                              # JSON-LD builders (no constants)
│   ├── analytics.ts                        # safeTrack() + typed events
│   ├── storage.ts                          # versioned localStorage helper (optional)
│   ├── share.ts                            # URL ↔ state codec (optional)
│   └── mdx.ts                              # loadPseoMdx (optional)
├── public/
│   ├── network.json                        # HUB
│   ├── ai-bots.json                        # HUB (also generated as ai-bots.txt)
│   ├── ads.txt                             # HUB OR GENERATED
│   ├── app-ads.txt                         # HUB OR GENERATED
│   ├── humans.txt                          # GENERATED (optional)
│   ├── .well-known/security.txt            # GENERATED (RFC 9116)
│   ├── og-default.png                      # SPOKE 1200×630 fallback
│   ├── favicon.ico                         # GENERATED from app/icon.svg
│   ├── favicon-16.png                      # GENERATED
│   ├── favicon-32.png                      # GENERATED
│   ├── icon-192.png                        # GENERATED
│   ├── icon-512.png                        # GENERATED
│   └── apple-touch-icon-180.png            # GENERATED
├── scripts/
│   ├── build-changelog.ts                  # git log → content/changelog.json
│   ├── build-llms-txt.ts                   # → public/llms.txt (in route handler)
│   ├── build-llms-full-txt.ts              # → public/llms-full.txt
│   ├── build-discovery.ts                  # ads/app-ads/security/humans + favicons
│   └── generate-favicon.ts                 # SVG → 16/32/192/512 + apple via sharp
├── tests/
│   ├── unit/**/*.test.ts                   # Vitest
│   └── e2e/**/*.spec.ts                    # Playwright
├── extension/                              # OPT-IN (features.extension)
├── .env.example                            # see §13
├── .gitattributes                          # LF normalization, binary lockfile
├── .gitignore                              # Next + Vercel + Playwright + LHCI
├── .nvmrc                                  # 22
├── .prettierrc.json
├── eslint.config.mjs
├── lighthouserc.json                       # mobile budgets: perf≥0.9, a11y/bp/seo≥0.95, CLS=0
├── next.config.ts                          # uses lib/csp.ts
├── package.json
├── playwright.config.ts
├── postcss.config.mjs
├── tsconfig.json
├── vercel.json                             # see §11
└── vitest.config.ts
```

Files prefixed `OPT-IN` ship only when the corresponding `siteConfig.features.*` flag is true.

---

## 4. The keystone — `lib/site-config.ts`

Every hardcoded brand string in a spoke must read from this object. Universal gap across all 5 audited spokes.

```ts
export const siteConfig = {
  // identity
  name: "<Site Name>",                    // brand name as shown to users
  shortName: "<ShortName>",               // ≤12 chars for manifest short_name
  domain: "example.com",                  // bare hostname
  url: "https://example.com",             // canonical absolute URL (no trailing slash)
  tagline: "Short value prop.",
  description: "Longer description for OG / metadata / llms.txt.",
  keywords: ["..."],                      // optional; root metadata

  // contact / legal
  supportEmail: "hello@example.com",
  trademarkDisclaimer: "...",             // optional; affiliate / fair-use

  // theme — token values, not class names; drives @theme + JSON-LD + OG colors
  theme: {
    colors: {
      bg: "#...",
      surface: "#...",
      accent: "#...",
      onBg: "#...",
      onAccent: "#...",
    },
    fontDisplay: "Inter",
    fontBody: "Inter",
    radiusCard: "0.75rem",
  },

  // brand mark
  brand: {
    markColor: "#...",                    // primary mark color (also used by app/icon.svg)
    markBgColor: "#...",
  },

  // navigation
  nav: {
    primary: [
      { href: "/", label: "Home" },
      // ...
    ],
    footer: {
      product: [/* { href, label } */],
      company: [/* ... */],
      legal: [/* ... */],
    },
    disclaimer: "Independent tool, not affiliated with ...",  // optional
  },

  // JSON-LD
  jsonLd: {
    type: "WebApplication",               // or SoftwareApplication, FinanceApplication
    operatingSystem: "Web",
    applicationCategory: "BusinessApplication",
    price: 0,
  },

  // GitHub
  repoUrl: "https://github.com/...",      // for changelog page footer

  // feature flags — drive opt-in template surfaces
  features: {
    embed: { enabled: false, route: "/embed/widget", params: [] },
    extension: { enabled: false, chromeWebStoreUrl: "" },
    proEnabled: false,                    // Stripe gating
    email: { enabled: false, leadMagnetName: "" },
    ads: {
      provider: "none",                   // none | adsense | mediavine | carbon
    },
    affiliate: {
      enabled: false,
      url: "",
      label: "",
      provider: "",                       // analytics tag
    },
    consent: { required: true },          // recommended on for EU traffic
    themeToggle: false,                   // dark/light toggle (optional)
  },

  // monetization (subset of features.*; declared concretely)
  monetization: {
    stripe: { priceIds: { monthly: "", yearly: "" } },
    lemonSqueezy: { storeId: "", productSlug: "" },
    gumroad: { productUrl: "", price: 0 },
  },

  // SEO verification — public, but env-driven so values never land in repo
  verification: {
    google: process.env.NEXT_PUBLIC_GOOGLE_SITE_VERIFICATION,
    bing: process.env.NEXT_PUBLIC_BING_SITE_VERIFICATION,
  },

  // security contact (RFC 9116)
  security: {
    contact: "mailto:security@example.com",
    expires: "2027-01-01T00:00:00Z",
  },
} as const;

export type SiteConfig = typeof siteConfig;
```

**Rule:** every hardcoded `name`, `domain`, `email`, `social handle`, `disclaimer`, or `nav link` in any other file is a P0 audit failure.

---

## 5. `@theme` design tokens (canonical names)

`app/globals.css` defines the canonical token namespace. Token *names* are network-wide; *values* are per-site (driven by `siteConfig.theme`).

```css
@theme {
  /* color */
  --color-bg: ...;
  --color-surface: ...;
  --color-accent: ...;
  --color-on-bg: ...;
  --color-on-accent: ...;
  --color-muted: ...;
  --color-border: ...;
  --color-success: ...;
  --color-danger: ...;

  /* typography */
  --font-display: ...;
  --font-body: ...;
  --text-display: 3.5rem;
  --text-h1: 2.25rem;
  --text-h2: 1.75rem;
  --text-h3: 1.375rem;

  /* spacing — 44px touch target floor */
  --spacing-touch: 2.75rem;

  /* radius */
  --radius-card: 0.75rem;
  --radius-pill: 9999px;

  /* shadow */
  --shadow-card: ...;
  --shadow-cta: ...;

  /* CLS-safe ad slot reserves (zero layout shift even pre-fill) */
  --ad-slot-banner-h: 90px;
  --ad-slot-rect-h: 250px;
  --ad-slot-leaderboard-h: 90px;

  /* utilities */
  --focus-ring: 2px solid var(--color-accent);
}
```

Components reference tokens via Tailwind v4 arbitrary syntax (`text-(--color-on-bg)`, `bg-(--color-surface)`). No raw hex literals in JSX or MDX components.

---

## 6. Required infrastructure files (the union)

| Path | Source | Spec |
|---|---|---|
| `app/robots.ts` | ENV + HUB | Returns `MetadataRoute.Robots`. Sitemap URL from `siteConfig.url`. AI bot allowlist (`GPTBot`, `ClaudeBot`, `Google-Extended`, `PerplexityBot`, `CCBot`, `Bingbot`) loaded from `public/ai-bots.json` (HUB-synced). |
| `app/sitemap.ts` | CONFIG + SPOKE | Multi-shard at >45k URLs. `lastmod` from `VERCEL_GIT_COMMIT_AUTHOR_DATE` for stable diffs. Includes static + pSEO + evergreen entries. |
| `app/llms.txt/route.ts` | CONFIG + SPOKE | Returns `text/plain`. 3h `Cache-Control`. ~12 KB. Lists primary surfaces + pSEO index. |
| `app/llms-full.txt/route.ts` | CONFIG + SPOKE | Returns `text/plain`. 3h cache. Long-form. MDX→text via `stripComponent` helper (promote to template util). |
| `app/ads.txt/route.ts` | HUB | Static text from `public/ads.txt` (HUB-synced) + `siteConfig.monetization` placeholder substitution. |
| `app/app-ads.txt/route.ts` | HUB | Mirrors ads.txt for in-app inventory; required by IAB even if no app. |
| `app/.well-known/security.txt` (or static `public/.well-known/security.txt`) | CONFIG | RFC 9116. `Contact: <siteConfig.security.contact>`, `Expires: <siteConfig.security.expires>`, `Preferred-Languages: en`. |
| `public/humans.txt` | GENERATED | Optional but cheap. Credits + last-updated. |
| `app/manifest.ts` | CONFIG | name, short_name, theme_color, background_color, icons[16,32,192,512]. `start_url` UTM-tagged. |
| `app/icon.svg` | SPOKE | 64×64 brand mark; brand color from `siteConfig.brand.markColor`. |
| `app/apple-icon.tsx` | CONFIG | 180×180 via `next/og` `ImageResponse`. |
| `app/opengraph-image.tsx` | CONFIG | 1200×630 via `next/og`. Edge runtime. State-encoded params allowed. |
| `app/twitter-image.tsx` | CONFIG | One-line re-export of opengraph-image. |
| `public/og-default.png` | SPOKE | 1200×630 PNG fallback for routes without dynamic OG. |
| `public/favicon.ico` + `favicon-16/32.png` + `icon-192/512.png` + `apple-touch-icon-180.png` | GENERATED | `scripts/generate-favicon.ts` runs sharp on `app/icon.svg`. Universal gap across all 5 audited spokes. |
| `vercel.json` | CONFIG | HSTS 2y, `X-Frame-Options: DENY` (except `/embed/*`), `Permissions-Policy` (camera/mic/geolocation =()), `Referrer-Policy: strict-origin-when-cross-origin`, font cache 1y immutable, `/llms.txt` 24h, `/embed` allows iframe. Apex ↔ www redirect. |
| `.gitattributes` | T1 | LF normalization for `*.ts`/`*.tsx`/`*.md`/`*.json`; binary `pnpm-lock.yaml` to prevent merge conflicts. |
| `.nvmrc` | T1 | `22`. |

---

## 7. Layout chrome contract

- **`SiteHeader`**: desktop nav at ≥640px from `siteConfig.nav.primary`. **Hamburger menu mandatory for mobile only** below 640px (zero-JS `<details>`, or `<dialog>`-based with focus trap + Escape-to-close + route-change-close). Brand mark from `<Wordmark>` reading `siteConfig.name`.
- **`SiteFooter`**: column array from `siteConfig.nav.footer`. Auto-year `new Date().getFullYear()`. Required blind footer link: **"Part of the Vertex Network"** → `/network` (the canonical hub-and-spoke entry point). Affiliate disclosure when `features.affiliate.enabled`.
- **`/network` page**: server component. Reads `public/network.json` (HUB). Filters out the current site by matching `siteConfig.url`. Renders `Property` cards. Page metadata mentions sister-site count, not specific names (avoid stale name lists). Includes `NetworkCollectionJsonLd`.
- **`Property` shape (locked across network):**
  ```ts
  type Property = {
    slug: string;        // e.g., "tokenmath"
    name: string;        // brand name
    domain: string;      // bare hostname
    url: string;         // absolute, no trailing slash
    tagline: string;     // ≤80 chars
    description: string; // ≤160 chars
    audience: string;    // who it's for
    tags: string[];
    status: "live" | "soon";
  };
  ```
- **Theme toggle**: optional. Use `localStorage` + `prefers-color-scheme` if enabled (`siteConfig.features.themeToggle`).
- **Cookie consent**: required when `siteConfig.features.consent.required` is true. Gates Clarity script loading. localStorage key namespaced as `${siteConfig.shortName}-consent-v1`.

### Required pages (every spoke ships all six)

The following routes are **mandatory** on every spoke regardless of compliance level. Missing any of them is a P0 audit failure.

| Route | Source | Spec |
|---|---|---|
| `/about` | SPOKE prose | Why this site exists, who built it. Renders `<AboutPageSchema>` JSON-LD when ad provider is Mediavine. Reads `siteConfig.name` / `siteConfig.supportEmail`. |
| `/contact` | CONFIG | Single email address surfaced from `siteConfig.supportEmail`, plus optional brief response-time copy. No form unless email capture is enabled. |
| `/privacy` | HUB MDX with `{{siteName}}` / `{{contactEmail}}` / `{{adProvider}}` placeholders, OR CONFIG-rendered | Lists every analytics + ad + monetization provider actually enabled on this site (driven by `siteConfig.features.*`). Sourced from `vertex-hub/content/legal/privacy.mdx` so legal updates fan out network-wide. |
| `/terms` | SPOKE (or HUB MDX template w/ overrides) | Product-specific usage terms. Use the hub MDX template as the starting point; override only where the product imposes specific rules (e.g., simulator/embed/spec-accuracy clauses). |
| `/changelog` | GENERATED from `content/changelog.json` | **Dates + titles only — no full release notes.** Each entry: `{ date: "YYYY-MM-DD", title: string }`. Script `scripts/build-changelog.ts` runs in `prebuild`, parsing `git log --pretty=format:'%ad|%s' --date=short` and emitting an idempotent, merge-safe JSON. The page renders a reverse-chronological list (one line per entry). Long-form release notes live in GitHub Releases (linked from the page header), not in the JSON. |
| `/network` | HUB | See above. The Vertex Network registry page. |

`siteConfig.nav.footer.legal` should link to all of: `/about`, `/contact`, `/privacy`, `/terms`, `/changelog`. The `/network` link sits in its own footer column or next to the "Part of the Vertex Network" attribution per existing convention.

**Changelog data shape (locked):**

```ts
// content/changelog.json
type ChangelogEntry = {
  date: string;   // "YYYY-MM-DD"
  title: string;  // commit subject; ≤80 chars; no body, no bullet list
};
type Changelog = ChangelogEntry[];  // sorted desc by date
```

Audit invariant: the page must not render any field beyond `date` and `title`. If a project surfaces additional commit body / author / hash, that's a DIVERGENT classification — flag for reconciliation.

---

## 8. SEO / GEO contract

- **Root metadata** (`app/layout.tsx`): `metadataBase`, `title.template = '%s | <name>'`, `title.default`, `description`, `keywords`, `openGraph`, `twitter` (`card: "summary_large_image"`), `verification: { google, other: { 'msvalidate.01' } }` from `siteConfig.verification`.
- **Per-page metadata**: every route exports `metadata` (or `generateMetadata`) with `alternates: { canonical }`.
- **JSON-LD**: emitted via `<JsonLd>` component. Universal types: `WebSite`, `Organization`, `BreadcrumbList`, `FAQPage`, `SoftwareApplication`/`WebApplication`/`FinanceApplication` (`@type` from `siteConfig.jsonLd.type`). `AboutPage` schema required if Mediavine ad network is enabled.
- **AI bot allowlist** (`robots.ts`): `GPTBot`, `ClaudeBot`, `Google-Extended`, `PerplexityBot`, `CCBot`, `Bingbot`, `Applebot-Extended`. Source: `public/ai-bots.json` (HUB).
- **`llms.txt` + `llms-full.txt`**: served as routes (3h cache) OR generated to `public/` at `prebuild`. MDX-to-plaintext stripper is shared util. Concise version ≤15 KB; full version ≤100 KB.
- **pSEO recipe** (universal high-leverage pattern, 5 steps):
  1. Define data source (`lib/<noun>-catalog.ts`) returning slug list with frontmatter shape (`slug`, `title`, `description`, `category`, `related[]`, `publishedAt`, custom fields).
  2. Add MDX directory `content/<noun>/<slug>.mdx` per slug.
  3. Add dynamic route `app/<noun>/[slug]/page.tsx` with `generateStaticParams` + `generateMetadata` (canonical, OG with state-encoded image URL) + `<MDX components={pSeoSlots} />`.
  4. Wire `app/sitemap.ts` to emit slug URLs.
  5. Wire `scripts/build-llms-full-txt.ts` to include catalog.

  Build-time invariants: meta-title ≤60 chars, meta-description ≤155 chars, FAQ Q-string dedup, ≥250 words body. Enforce in Vitest.

---

## 9. Analytics contract

| Tool | When | How |
|---|---|---|
| Vercel Analytics | always | `<Analytics />` in root layout |
| Vercel Speed Insights | always | `<SpeedInsights />` in root layout |
| Microsoft Clarity | when `NEXT_PUBLIC_CLARITY_PROJECT_ID` set | Loaded via `<ClarityScript />`, **gated by consent** when consent provider enabled |
| GSC verification | when `NEXT_PUBLIC_GOOGLE_SITE_VERIFICATION` set | Emits `<meta name="google-site-verification">` from root metadata |
| Bing verification | when `NEXT_PUBLIC_BING_SITE_VERIFICATION` set | Emits `<meta name="msvalidate.01">` from root metadata |
| Plausible | optional, when `NEXT_PUBLIC_PLAUSIBLE_DOMAIN` set | `lazyOnload` script |
| GA4 | optional, when `NEXT_PUBLIC_GA_ID` set | gtag wrapper in `lib/analytics.ts` |

Custom analytics events live in `lib/analytics.ts` with a `safeTrack(event, props)` wrapper that no-ops when no provider is initialized. Required network-wide event: **`vertex_footer_opened`** (fires when user clicks "Part of the Vertex Network" link).

---

## 10. Monetization contract

- **Ad provider switch**: `NEXT_PUBLIC_AD_PROVIDER` ∈ `{none, adsense, mediavine, carbon}`. `<AdsProviderScript>` loads the matching provider SDK. `<AdSlot>` renders provider-specific markup or returns `null` when `none`.
- **CLS-safe heights**: `--ad-slot-*-h` reserved in `@theme` so layout doesn't shift when ads fill.
- **Single affiliate slot principle**: `siteConfig.features.affiliate` carries one `{url, label, provider}`. Multiple affiliates allowed but documented as discouraged (per tokenmath's network discipline).
- **Stripe gating**: opt-in via `siteConfig.features.proEnabled`. Stripe SDK in `app/api/stripe/{checkout,welcome,portal,webhook}/route.ts`. License token (HMAC-signed, 24h) stored in localStorage with namespaced key `${shortName}-license-v1`. `cs_cid` long-lived HTTP-only cookie ties browser → Stripe customer.
- **Lemon Squeezy / Gumroad**: opt-in via `siteConfig.monetization.{lemonSqueezy,gumroad}`. URL-builder helper in `lib/checkout.ts`.
- **Email capture (Resend)**: opt-in via `siteConfig.features.email`. `app/api/subscribe/route.ts` runs on Edge, validates email, forwards to Resend Audience API (treats 409 as success).
- **Embed widget**: opt-in via `siteConfig.features.embed.enabled`. `next.config.ts` adds `frame-ancestors *` CSP header for `/embed/*` only. `<EmbedSnippet>` provides copy-to-clipboard iframe code.

---

## 11. CI/CD contract

### Reusable workflows (HUB-owned, called from spokes)

```yaml
# .github/workflows/ci.yml in spoke
name: CI
on: [pull_request, push]
jobs:
  ci:
    uses: vertex-network/.github/workflows/ci.yml@v1
    with:
      node-version: "22"
```

Hub workflow runs: `pnpm install` → typecheck → lint → vitest → build → Lighthouse CI (mobile budgets perf≥0.9, a11y/bp/seo≥0.95, **CLS=0**).

Tag with `@v1` so breaking changes don't auto-roll spokes.

### Per-spoke workflows (committed locally)

| Workflow | Purpose |
|---|---|
| `sync-from-hub.yml` | `repository_dispatch` listener; on hub push, pulls updated `network.json`/`ads.txt`/`ai-bots.json` and opens auto-merge PR. |
| `sitemap-ping.yml` | Post-deploy: POST sitemap to GSC IndexNow + Bing IndexNow. |
| `screenshot-bot.yml` (optional) | On main push, capture homepage screenshot, commit to hub `screenshots/{domain}.png` for `/network` previews. |
| `lighthouse.yml` | LHCI on every preview deploy. Calls hub reusable workflow. |
| `release.yml` (optional) | Auto-tag on conventional commits. |

### CSP builder (`lib/csp.ts`)

```ts
export function buildCSP(providers: {
  vercelAnalytics?: boolean;
  adsense?: boolean;
  mediavine?: boolean;
  carbon?: boolean;
  clarity?: boolean;
  plausible?: boolean;
}): string { /* compose script-src/connect-src/etc conditionally */ }
```

Drives `next.config.ts` headers. Avoids shipping every spoke with the union allowlist.

---

## 12. Hub artifacts (lives in `vertex-network` / `vertex-hub` repo)

```
vertex-hub/
├── config/
│   ├── network.json                 # canonical Property[] registry
│   ├── ai-bots.json                 # AI bot allowlist
│   ├── ads-publishers.json          # ads.txt publisher lines
│   ├── env.template.json            # required env vars per spoke
│   └── csp-providers.json           # CSP allowlist deltas when providers change
├── content/
│   ├── network-philosophy.mdx       # "What ties them together" prose
│   ├── llms-boilerplate.md          # llms.txt header/footer
│   ├── trademark-disclaimers.md     # shared legal copy
│   └── legal/
│       ├── privacy.mdx              # MDX with {{siteName}} / {{contactEmail}} placeholders
│       ├── terms.mdx                # (opt-in per site)
│       └── refund.mdx               # (Stripe sites)
├── crawler/
│   ├── ads.txt
│   ├── app-ads.txt
│   ├── security.txt                 # template (per-site Contact substituted)
│   └── humans.txt                   # template
├── templates/
│   ├── workflows/                   # canonical reusable workflows
│   ├── .gitignore
│   ├── .env.example
│   ├── tsconfig.json
│   ├── eslint.config.mjs
│   └── .prettierrc.json
├── packages/                        # OPTIONAL future @vertex/* npm packages
│   ├── storage/                     # versioned localStorage helper
│   ├── changelog/                   # git log → JSON
│   ├── seo/                         # JsonLd + llms.txt renderers
│   ├── analytics/                   # safeTrack + Clarity
│   ├── ads/                         # AdSlot + provider scripts
│   └── shell/                       # SiteHeader/SiteFooter taking siteConfig
├── scripts/
│   ├── propagate.ts                 # fans repository_dispatch to all spokes
│   └── create-spoke.ts              # `npx create-vertex-site`
├── docs/
│   ├── SITE_LAUNCH_CHECKLIST.md     # 30-step ops runbook (manual steps)
│   └── ARCHITECTURE.md
└── .github/workflows/
    ├── propagate.yml                # on push to main → repository_dispatch fan-out
    ├── network-validate.yml         # JSON schema validation
    └── ci.yml                       # reusable workflow consumed by spokes
```

**Sync mechanism (recommended).** PR-based fan-out via `repository_dispatch`:

1. Hub `propagate.yml` triggers on push to `main` to any synced file.
2. Reads `spokes.json` (list of spoke repos).
3. For each spoke, opens a PR via `peter-evans/create-pull-request` updating the corresponding files.
4. Spoke CI gates the merge; auto-merge if green.

Alternative for low-risk assets (build-time fetch in `prebuild`): simpler but couples build to hub availability. Not recommended for production sites.

---

## 13. Per-site env var canonical list (the contract)

```sh
# Identity (required)
NEXT_PUBLIC_SITE_NAME=
NEXT_PUBLIC_SITE_SHORT_NAME=
NEXT_PUBLIC_SITE_DOMAIN=
NEXT_PUBLIC_SITE_URL=
NEXT_PUBLIC_SITE_DESCRIPTION=
NEXT_PUBLIC_SITE_CONTACT_EMAIL=

# SEO verification
NEXT_PUBLIC_GOOGLE_SITE_VERIFICATION=
NEXT_PUBLIC_BING_SITE_VERIFICATION=

# Analytics
NEXT_PUBLIC_VERCEL_ANALYTICS=1
NEXT_PUBLIC_CLARITY_PROJECT_ID=
NEXT_PUBLIC_PLAUSIBLE_DOMAIN=        # optional
NEXT_PUBLIC_GA_ID=                   # optional

# Ad provider switch
NEXT_PUBLIC_AD_PROVIDER=none         # none | adsense | mediavine | carbon
NEXT_PUBLIC_ADSENSE_CLIENT_ID=
NEXT_PUBLIC_MEDIAVINE_SITE_ID=
NEXT_PUBLIC_CARBONADS_SERVE_ID=
NEXT_PUBLIC_CARBONADS_PLACEMENT=

# Affiliate
NEXT_PUBLIC_AFFILIATE_ENABLED=0
NEXT_PUBLIC_AFFILIATE_URL=
NEXT_PUBLIC_AFFILIATE_LABEL=
NEXT_PUBLIC_AFFILIATE_PROVIDER=

# Email capture
RESEND_API_KEY=                      # server-side only
RESEND_AUDIENCE_ID=

# Stripe (Pro tier, opt-in)
STRIPE_SECRET_KEY=
STRIPE_WEBHOOK_SECRET=
NEXT_PUBLIC_PRO_ENABLED=0
NEXT_PUBLIC_STRIPE_PUBLISHABLE_KEY=
NEXT_PUBLIC_STRIPE_PRICE_MONTHLY=
NEXT_PUBLIC_STRIPE_PRICE_YEARLY=

# Lemon Squeezy / Gumroad (opt-in)
NEXT_PUBLIC_LEMON_SQUEEZY_STORE_ID=
NEXT_PUBLIC_GUMROAD_PRODUCT_URL=
NEXT_PUBLIC_GUMROAD_PRICE=

# Embed
NEXT_PUBLIC_EMBED_ENABLED=0
```

Producer column: identity + verification = human (per-site); analytics/ad/affiliate IDs = human (Vercel project settings); secrets (Stripe, Resend) = Vercel encrypted env.

---

## 14. Manual steps remaining (launch checklist)

These cannot be automated end-to-end. Lives in `vertex-hub/docs/SITE_LAUNCH_CHECKLIST.md`:

1. Register domain (Namecheap).
2. Configure DNS A/CNAME (can semi-automate via Namecheap API in scaffolder).
3. Create Vercel project (Vercel API automatable in scaffolder).
4. Seed Vercel env vars (Vercel API automatable).
5. Configure apex ↔ www redirect (`vercel.json`).
6. **Google Search Console**: claim site, submit sitemap, copy verification token to env.
7. **Bing Webmaster**: claim site, submit sitemap, copy verification token to env.
8. AdSense site claim (Mediavine application: traffic threshold required).
9. Chrome Web Store submission (if extension; opt-in).
10. Social profile registration (Twitter handle reservation).
11. Add spoke to `vertex-hub/config/network.json` and `spokes.json`; merge to fan out to siblings.

---

## 15. Compliance levels

The audit prompt grades spokes against one of three levels:

| Level | Required |
|---|---|
| **MINIMAL** | siteConfig.ts, hamburger header, **all six required pages (`/about`, `/contact`, `/privacy`, `/terms`, `/changelog`, `/network`)**, `/network` reading hub JSON, robots/sitemap/llms.txt/llms-full.txt, ads.txt + app-ads.txt + security.txt, Vercel Analytics, GSC + Bing verification env-driven, multi-size favicon set, `vercel.json` with security headers, CI workflow |
| **STANDARD** (default) | MINIMAL + Speed Insights + Clarity (env-gated) + cookie consent (when EU) + dynamic OG route + JSON-LD universal types + pSEO pattern + reusable CI via `uses:` + `sync-from-hub.yml` + Lighthouse CI + auto-changelog from git |
| **PREMIUM** | STANDARD + Lighthouse budgets enforced + sitemap-ping on deploy + broken-link cron + a11y axe-core CI + OG snapshot test + screenshot bot for `/network` previews |

Default audit grade: **STANDARD**.
