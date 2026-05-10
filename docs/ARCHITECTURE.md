# ARCHITECTURE — how the hub-and-spoke loop works

In one diagram:

```
┌─────────────────────────────────────┐
│  vertex-network-hub  (this repo)    │
│                                     │
│   config/network.json   ← edit here │
│   config/ai-bots.json               │
│   crawler/{ads,app-ads,humans}.txt  │
│   content/legal/*.mdx               │
│                                     │
│           push to main              │
│                ↓                    │
│   .github/workflows/propagate.yml   │
│   reads config/spokes.json          │
│                ↓                    │
└────────────────┼────────────────────┘
                 │ repository_dispatch
                 │ event: vertex-hub-sync
                 │ (one per spoke, fan-out)
                 ↓
   ┌─────────────┴──────────────┬───────────┬───────────┐
   ↓                            ↓           ↓           ↓
┌──────────┐              ┌──────────┐  ┌──────────┐  ...
│ tokenmath│              │captionsnap│ │ shopifont│
│          │              │           │ │          │
│ .github/workflows/      │           │ │          │
│ sync-from-hub.yml       │ (same)    │ │ (same)   │
│        ↓                │           │ │          │
│ curl raw.github →       │           │ │          │
│   public/network.json   │           │ │          │
│   public/ai-bots.json   │           │ │          │
│   (NETWORK-WIDE ONLY)   │           │ │          │
│        ↓                │           │ │          │
│ open auto-merge PR      │           │ │          │
│        ↓                │           │ │          │
│ CI passes → merge       │           │ │          │
│        ↓                │           │ │          │
│ Vercel auto-deploys     │           │ │          │
└──────────┘              └──────────┘ └──────────┘
```

## Two consumption models for spokes

**Model A — audit reference (zero install).** Spokes never copy hub files in. The audit prompt at [`_canonical-audit-prompt.md`](_canonical-audit-prompt.md) tells Claude to fetch the hub remotely over WebFetch / raw GitHub URLs, then grade the local spoke against what it finds. This is the primary loop and works for any project (existing, in-progress, or empty).

**Model B — auto-sync (one-file install).** A spoke that wants hub edits to land automatically installs [`templates/spoke/.github/workflows/sync-from-hub.yml`](../templates/spoke/.github/workflows/sync-from-hub.yml). On hub push, it pulls updated `network.json`/`ai-bots.json`/`ads.txt`/etc. and opens an auto-merge PR. That's the only file the spoke needs.

Most spokes start with Model A, then add Model B for individual files (just `network.json`, say) once the audit confirms the value.

## Why this shape

**Hub-and-spoke beats monorepo for indie networks.** Each site keeps its own deploy cadence, its own analytics IDs, its own incident blast radius. But the network registry, ads.txt, AI-bot allowlist, and legal boilerplate have one source of truth. Edit once, propagates everywhere.

**Why PR-based fan-out (not build-time fetch).** Every change is auditable per-spoke. Spokes keep building if the hub is briefly down. CI gates each sync, so a malformed `network.json` can't take down 30 sites in one shot.

**Why a reusable workflow at `<owner>/<repo>/.github/workflows/spoke-ci.yml@v1` (not a separate `<owner>/.github` repo).** Fewer moving parts. Org-default `.github` repos are convenient but add a second repo to maintain. The reusable workflow lives here in the hub and is consumed by `uses:` from spokes.

## What lives where

| Concern | Where |
|---|---|
| Network registry (the `Property[]`) | `config/network.json` |
| AI bot allowlist for `robots.ts` | `config/ai-bots.json` |
| Spoke registry (who gets dispatched to) | `config/spokes.json` |
| Files served at well-known paths on every spoke | `crawler/` |
| Legal boilerplate with `{{placeholders}}` | `content/legal/` |
| The "what ties them together" prose | `content/network-philosophy.mdx` |
| Hub's own CI for fan-out + JSON validation | `.github/workflows/propagate.yml`, `network-validate.yml` |
| The reusable workflow spokes consume | `.github/workflows/spoke-ci.yml` (referenced as `@v1`) |
| Two **optional** workflow files spokes can install (sync listener + CI stub) | `templates/spoke/` |
| Spec, audit prompt, synthesis | `docs/_*.md` |
| One-time setup runbook | `docs/SETUP.md` (read first) |
| Per-spoke launch runbook | `docs/SITE_LAUNCH_CHECKLIST.md` |

## Sync triggers

`propagate.yml` fires on push to `main` when any of these paths change:

- `config/network.json`
- `config/ai-bots.json`
- `config/env.template.json`
- `crawler/**`
- `content/**`

It can also be triggered manually from the Actions tab via `workflow_dispatch`.

Spokes have a daily 06:00 UTC cron in `sync-from-hub.yml` as a belt-and-suspenders safety net in case a dispatch is missed.

## What gets synced — and what doesn't

The defaults are **maximally conservative**: only files that are genuinely identical on every spoke get pulled. Everything else stays under the spoke's control.

**Synced by default (overwritten on every hub propagation):**

| Hub source | Spoke destination |
|---|---|
| `config/network.json` | `public/network.json` |
| `config/ai-bots.json` | `public/ai-bots.json` |

**NOT synced (the spoke owns these; hub never touches them):**

- `public/ads.txt`, `public/app-ads.txt` — different AdSense/Mediavine publisher per site
- `public/humans.txt` — per-site credits
- `public/.well-known/security.txt` — per-site security contact
- `siteConfig.ts` — per-site brand, colors, nav, theme
- `app/icon.svg`, `public/og-default.png` — brand mark + OG fallback
- All product code (calculators, generators, pSEO content)
- All env vars (analytics IDs, ad provider IDs, Stripe keys, etc.)
- The hero `app/page.tsx`

**Opt-in to additional syncing:** if a particular spoke has no per-site customization on `ads.txt`/`app-ads.txt`/`humans.txt` and wants the hub's version, uncomment the corresponding `curl` line in that spoke's `sync-from-hub.yml`. This is a per-spoke choice — uncommenting in one spoke doesn't affect others.

## Versioning the reusable workflow

`spoke-ci.yml` is referenced by spokes as `@v1`. To change it:

- **Bug fixes / additive changes** → push to `main`, then move the `v1` tag (`git tag -f v1 && git push -f origin v1`). Spokes pick it up on their next run.
- **Breaking changes** (input renames, behavior changes) → create a `v2` tag. Migrate spokes one at a time. Keep `v1` working until all spokes have moved.

## Failure modes and what happens

| Failure | What happens | Mitigation |
|---|---|---|
| PAT expires or is revoked | `propagate.yml` fails; no spokes get dispatched | Daily cron in spokes' `sync-from-hub.yml` still pulls latest |
| Bad JSON in `network.json` | `network-validate.yml` blocks the PR | Fix and re-push |
| Spoke missing `public/` dir | `curl` writes to it; PR opens fine | Spokes should always have `public/` (Next.js convention) |
| Hub repo briefly unavailable | Spokes building from cache continue working | Ephemeral; next sync recovers |
| Two sync PRs open at once | Second one supersedes the first via `branch: hub-sync` | `peter-evans/create-pull-request` handles this |
