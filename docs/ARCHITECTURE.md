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
│   public/ads.txt        │           │ │          │
│   public/app-ads.txt    │           │ │          │
│   public/humans.txt     │           │ │          │
│        ↓                │           │ │          │
│ open auto-merge PR      │           │ │          │
│        ↓                │           │ │          │
│ CI passes → merge       │           │ │          │
│        ↓                │           │ │          │
│ Vercel auto-deploys     │           │ │          │
└──────────┘              └──────────┘ └──────────┘
```

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
| Files spokes copy in (workflows, vercel.json, etc.) | `templates/spoke/` |
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

## What stays in the spoke (not synced)

- `siteConfig.ts` — per-site brand, colors, nav.
- `app/icon.svg` and `public/og-default.png` — brand mark + OG fallback.
- All product code (calculators, generators, pSEO content).
- All env vars (analytics IDs, ad provider IDs, Stripe keys).
- The hero `app/page.tsx`.

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
